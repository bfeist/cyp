# Clearinghouse API Architecture (Dog Match Platform)

_Last updated: 2026-03-02 (updated with deep research run findings from Briefs 2, 3, 7)_

> **Research findings incorporated:** connector priority order confirmed (Brief 2), Canada vendor stack confirmed (Brief 3), Selenium/queue/provenance architecture confirmed (Brief 7). See `00-executive-summary.md` for full decisions.

## Objective

Build a platform that acts as a **data clearinghouse** across three pillars:

1. Dog breeds and attributes
2. Adoption/purchase availability
3. Human profile traits for matchmaking

This architecture assumes a mixed ingestion strategy: **official APIs + partner feeds + large-scale Selenium-based web extraction** where strategically useful.

## Product-Level Architecture (High Level)

## Layer 1 — Source Acquisition

Connectors ingest data from:

- Breed registries and standards bodies
- Adoption aggregators and shelter platforms
- Breeder marketplaces and directories
- Human-profile interaction surfaces (app behavior, onboarding, preference signals)

Connector types:

- `api_connector` — REST/GraphQL/vendor SDK
- `feed_connector` — CSV/JSON/XML/SFTP dump
- `selenium_connector` — browser automation extraction for dynamic pages
- `manual_curation_connector` — analyst-entered enrichment

## Layer 2 — Normalization + Identity Resolution

- Field mapping into canonical schema
- Unit and taxonomy standardization
- Entity resolution:
  - `BreedIdentity`
  - `OrganizationIdentity`
  - `ListingIdentity`
  - `UserProfileIdentity`
- Duplicate collapse using deterministic + probabilistic matching

## Layer 3 — Data Quality + Provenance

- Per-field provenance records
- Confidence scoring
- Freshness timestamps
- Contradiction detection (e.g., listing says "good with cats" in one source and "not cat friendly" in another)

## Layer 4 — Intelligence Services

- Trait extraction from free text
- Breed-risk/fit scoring
- Human profile embeddings and preference models
- Matching and re-ranking services

## Layer 5 — API Clearinghouse Surface

Expose your own APIs:

- `GET /breeds`
- `GET /breed-profiles/{id}`
- `GET /dogs/available`
- `GET /organizations`
- `POST /profiles`
- `POST /match/breed`
- `POST /match/listings`
- `GET /explanations/{match_id}`

These APIs are the productized output of all upstream ingestion complexity.

---

## Selenium-Centric Acquisition Design

## Why Selenium here

Many valuable sources are JS-heavy, paginated, and anti-static-fetch. Selenium is appropriate for:

- dynamic DOM rendering
- interactions (filters, infinite scroll, detail modals)
- authenticated workflows where permitted
- visual/behavioral regression checks of extraction selectors

## Selenium Extractor Pipeline (updated from Brief 7)

### Architecture overview

```
Control Plane
  Source Registry + Policy Engine (source metadata, tier, schedule, selector spec versions)
  Workflow Orchestrator (daily schedules, backfills, retries) → emits SourceRun jobs

Job Queue / Router
  Priority lanes (P0/P1/P2) | per-source concurrency caps | DLQ / poison isolation

Data Plane
  Tiered Acquisition Workers (Kubernetes deployments; HPA-scaled)
    ├── API/Feed workers
    ├── Selenium scrape workers
    └── Manual task hooks
  Browser Execution Fabric (Selenium Grid: Router → New Session Queue → Distributor → Nodes)
  Artifact + Provenance Store (object storage: WARC/HTML, screenshots, HAR, run manifests)
  Normalize + Validate + Dedupe → Clearinghouse Data Stores (operational DB + warehouse/lake)
  Observability (metrics + tracing + logs + breakage detection + canary monitors)
```

### SourceRun contract (minimum fields)

- `source_id`, `tier`, `schedule`, `rate_limit_budget`
- `extraction_spec_id` (versioned; treat selector changes as releases)
- `run_type` (incremental / full)
- `priority` (P0 / P1 / P2)

### Queue design

- At-least-once delivery; design all writes as idempotent (deterministic key: `source_id + native_id + observation_date`)
- Visibility timeout must exceed worst-case run time
- DLQ for repeated failures; isolate poison sources

### Breakage detection layers

1. Unit tests against stored HTML fixtures (pre-merge)
2. Canary runs on small segment before broad rollout (treat selector changes like software releases)
3. Synthetic monitors: scripted Selenium transactions on fixed schedule
4. Production alerts: selector exception rate, record count drops, schema coverage regressions

### Explicit waits (required)

All Selenium workers must use explicit waits (poll until condition satisfied), not sleep timers, for dynamic DOM synchronization.

## Recommended technical stack

- Selenium Grid (containerized) for horizontal browser scaling
- Chrome + Firefox runners
- Headless default, headed fallback for fragile flows
- Queue orchestration (e.g., Redis queue/Celery or cloud queue)
- Proxy/IP strategy where contractually allowed
- Selector configuration registry per source

## Data extraction artifacts to persist (confirmed by Brief 7)

Every SourceRun must produce a **run manifest** containing:

- `source_id`, `run_id`, timestamps, worker identity, browser/driver versions, IP/proxy identity, orchestrator job ID, ExtractionSpec version, code version

Plus blobs in object storage:

- **WARC/HTML snapshot** — ISO standard for web crawl archiving; essential for replaying regressions
- **Screenshots** — at minimum one final-state screenshot; step-level on failures
- **HAR log** — structured HTTP archive for debugging hidden XHR/GraphQL endpoints
- **Logs** — centralized, integrity-controlled, with explicit retention policy

## Anti-breakage controls

- selector fallback chains
- source-specific contract tests
- canary runs every few hours
- automatic incident flags on extraction drop > threshold

---

## Ingestion Governance Model

## Source policy tiers (confirmed and refined by Brief 7)

| Tier   | Definition                                                    | Examples                                                      | Notes                               |
| ------ | ------------------------------------------------------------- | ------------------------------------------------------------- | ----------------------------------- |
| Tier A | Official or partner APIs                                      | RescueGroups v5, ShelterLuv API, ShelterBuddy API             | Highest reliability, lowest compute |
| Tier B | Structured partner exports, FTP/SFTP feeds, syndication dumps | Petstablished FTP AutoUpload, PetPoint webservices activation | Moderate reliability, low compute   |
| Tier C | Selenium/browser automation for dynamic sites                 | AKC Marketplace, breeder directories, dynamic portals         | High compute, highest breakage rate |
| Tier D | Manual capture                                                | CAPTCHA-blocked or legally restricted sources                 | Human-assisted only                 |

**Promotion/demotion rules (confirmed by Brief 7):**

- Promote C → A/B when stable hidden JSON/GraphQL endpoints are discovered via HAR inspection
- Demote A/B → C only when API/feed coverage drops below threshold and policy permits scraping
- Demote C → D when automation is persistently blocked (403/429 loops, interstitial challenges)

## Confirmed connector priority order (from Brief 2)

1. RescueGroups — REST API + FTP download, Tier A
2. ShelterLuv — API key, Tier A
3. ShelterBuddy — REST API with incremental sync, Tier A
4. PetPoint/24Pet/24Petconnect — vendor-mediated, Tier B/A depending on agreement
5. Petstablished — API + FTP export, Tier B
6. Chameleon — Crystal Report + SFTP, Tier B (phase 2)
7. Shelter Pro, ASM — file-based, Tier B long tail

## For each source, maintain

- usage basis (API key, agreement, public page)
- allowed data classes
- retention limits
- refresh frequency
- kill switch owner

## Operational controls

- per-source rate limiter
- per-source stop switch
- legal/ops approval metadata
- full crawl audit trail

---

## Canonical Domain Model Extension (3-Pillar)

Add to prior schema docs:

- `HumanProfile`
- `HumanSignalEvent`
- `ProfileEmbedding`
- `PreferenceDrift`
- `MatchOutcomeFeedback`

This allows the system to move from static questionnaire logic to adaptive preference inference.

---

## Matching Engine Architecture

## Stage 1: Candidate generation

- Breed shortlist from hard constraints
- Listing shortlist by geography/channel availability

## Stage 2: Scoring

- Breed fit score
- Human lifestyle compatibility score
- Individual dog compatibility score
- Logistics score (distance, process friction, budget)

## Stage 3: Explanation layer

Return feature-attribution-style explanation, e.g.:

- "High match due to moderate exercise alignment and strong child compatibility"
- "Lowered due to grooming tolerance mismatch"

## Stage 4: Feedback loop

Use explicit and behavioral outcomes to recalibrate profile and ranking weights.

---

## Platform APIs You Become Responsible For

## Internal data APIs

- raw ingestion records
- normalized entities
- provenance endpoints
- source health endpoints

## External product APIs

- search/filter endpoints
- match scoring endpoints
- recommendation explanation endpoints

## Admin APIs

- source status and pause/resume
- data quality dashboards
- model/version management

---

## Security, Privacy, and Compliance Design

## Security

- field-level encryption for sensitive profile fields
- role-based access control across ingestion/admin/matching
- secrets vault for source credentials

## Privacy

- purpose-limited storage for human signals
- profile deletion workflow
- event retention windows
- explainability and user correction paths

## Compliance operations

- source removal/takedown pipeline
- provenance and audit exports
- configurable jurisdictional policy rules

---

## Operational KPIs (Golden Signals mapped to ingestion — from Brief 7)

| Signal     | What to measure                                                                 |
| ---------- | ------------------------------------------------------------------------------- |
| Latency    | Queue wait time; SourceRun duration; Grid session acquisition time              |
| Traffic    | Runs/day, pages/day, records/day per tier                                       |
| Errors     | Per-source failure rate, selector exception rate, HTTP 429/403/5xx distribution |
| Saturation | Browser node utilization, pending session queue depth, worker CPU/memory        |

## MVP Build Sequence

1. Implement 2-3 `Tier A/B` connectors + 1 Selenium connector.
2. Build canonical write path + provenance.
3. Release read-only clearinghouse API (`breeds`, `dogs/available`, `organizations`).
4. Add baseline human profile questionnaire + behavior event logging.
5. Launch v1 match endpoint with explanation output.
6. Add adaptive profile updates from user interaction outcomes.

## What to prototype immediately

- Selenium connector template with selector config abstraction
- unified `SourceAttribution` table as non-optional write dependency
- model-agnostic `MatchScore` service with versioned scoring recipes
