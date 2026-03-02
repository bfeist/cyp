# Clearinghouse API Architecture (Dog Match Platform)

_Last updated: 2026-03-01_

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

## Selenium Extractor Pipeline

1. `discover_job` — source-specific URL and filter seeds
2. `render_job` — open browser session, run navigation script
3. `extract_job` — parse DOM snapshots into structured records
4. `enrich_job` — classify text traits and normalize fields
5. `publish_job` — write raw + canonical payloads
6. `audit_job` — store screenshot, HTML hash, selector success stats

## Recommended technical stack

- Selenium Grid (containerized) for horizontal browser scaling
- Chrome + Firefox runners
- Headless default, headed fallback for fragile flows
- Queue orchestration (e.g., Redis queue/Celery or cloud queue)
- Proxy/IP strategy where contractually allowed
- Selector configuration registry per source

## Data extraction artifacts to persist

- raw HTML snapshot hash
- extracted JSON payload
- screenshot evidence
- extractor version
- selector set version
- runtime metrics (latency, retries, blocks)

## Anti-breakage controls

- selector fallback chains
- source-specific contract tests
- canary runs every few hours
- automatic incident flags on extraction drop > threshold

---

## Ingestion Governance Model

## Source policy tiers

- `Tier A` — official API/feed with explicit usage rights
- `Tier B` — partnership/contractual export
- `Tier C` — Selenium/web extraction with approved risk posture
- `Tier D` — manual curation only (no automated pull)

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
