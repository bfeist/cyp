# Executive Summary — CYP Platform

_Last updated: 2026-03-02_

---

## What we are building

**CYP (Customize Your Pet)** is a dog-owner matchmaking platform that helps people find the right dog — adopted or purchased — and helps shelters and rescues reduce return rates by making better matches in the first place.

The core problem: most adoption and purchase decisions are made on appearance and proximity. When the match is wrong — wrong energy level, wrong compatibility profile, wrong household fit — the dog is returned. Return rates at US shelters are estimated at 10–20% within the first 6 months, with behavior problems and expectation mismatch as the leading causes.

CYP's answer: build the data infrastructure the market is missing, then put intelligent matching on top of it.

### Three-pillar data foundation

```
  Pillar 1                    Pillar 2                   Pillar 3
  BREED                       INDIVIDUAL DOG             HUMAN PROFILE
  ──────────────────────────  ─────────────────────────  ──────────────────────────
  Breed-level traits          Per-dog behavioral         Owner constraint gates,
  (temperament, energy,       assertion model extracted  preference signals,
  health predispositions,     from shelter text +        psychometric patterns,
  grooming, size ranges)      structured notes           and outcome feedback loops
```

No current product has all three at depth. Most tools have Pillar 1 at surface level and skip Pillars 2 and 3 entirely.

### The clearinghouse model

CYP does not rely on a single data source. The adoption and shelter landscape is fragmented across thousands of organizations with no national index. The platform builds its own unified API by acting as a **data clearinghouse**: ingesting from shelter software systems (RescueGroups, ShelterLuv, ShelterBuddy, PetPoint), breed registries (AKC, CKC, CHIC/OFA), breeder marketplaces (Good Dog), and individual rescue websites — normalizing, deduplicating, and enriching everything into a canonical schema. Every field carries provenance: source, timestamp, confidence, and freshness.

### The matching model

Matching is not a filter. It is a scored, multi-dimensional recommendation system that improves over time:

- **v1**: Hard constraint gates (allergies, other pets, housing) + 5-component interpretable fit score from a structured onboarding questionnaire
- **v2**: ML learning-to-rank on implicit interaction signals (views with position logging, saves, dismissals, applications, adoption outcomes) with inverse propensity scoring for bias correction
- **v3**: Adaptive psychometric elicitation (pairwise micro-choices, conversational prompts mapped to ontology nodes, contextual bandit exploration) with post-adoption feedback loops informing retention risk models

The core success metric is **match retention at 30 and 90 days post-adoption**, not click-through rate.

---

## Current state (March 2026)

- Architecture fully designed across all three pillars; all 7 deep research briefs completed and integrated
- No application code written yet; this doc pack is the pre-build foundation
- MVP scope: US adoption ingestion (RescueGroups connector), v1 fit score model, event logging schema, canonical data schema
- Canada is Wave 2; breeder quality scoring is a parallel track to adoption

---

## Three Pillars: High-Level Status

### Pillar 1 — Breed Intelligence

Breed-level trait data is available but requires careful sourcing. AKC/CKC provide breed standards and registry metadata. CHIC and OFA hold the best health screening data but license restrictions require a partnership path — bulk scraping is explicitly prohibited. The canonical `Breed` entity covers physical dimensions, energy/trainability/temperament scores aggregated from registries, coat attributes, and a curated temperament summary. A separate `BreedHealthProfile` entity links to condition-specific screening recommendations and severity weights.

Key risk: AKC and OFA explicitly prohibit automated extraction. Treat both as contract-required integrations.

→ _Details in [`02-source-catalog.md`](02-source-catalog.md) and [`05-access-legal-scraping-guidelines.md`](05-access-legal-scraping-guidelines.md)._

### Pillar 2 — Individual Dog Availability + Behavioral Data

The adoption landscape is operationally regional and decentralized. There is no government-run national index in the US or Canada. National-scale visibility is created by private aggregators, but the actual source of truth is the shelter management system (SMS) used by each organization.

**Critical finding**: Petfinder's public API was decommissioned on 2025-12-02. Not a viable data source. Adopt-a-Pet APIs are partner-gated. The confirmed MVP spine is **RescueGroups v5**, with ShelterLuv, ShelterBuddy, and PetPoint/24Pet as the follow-on connector layer.

Connector priority:

| Priority | Platform         | Why                                                                 | Integration surface |
| -------- | ---------------- | ------------------------------------------------------------------- | ------------------- |
| 1        | RescueGroups     | 6,200+ partner orgs, v5 REST API active                             | REST API + FTP      |
| 2        | ShelterLuv       | 50-state network, behavioral attributes aligned to schema           | API key + polling   |
| 3        | ShelterBuddy     | Incremental sync (`UpdatedSinceUtc`), behavior assessment endpoints | REST API            |
| 4        | PetPoint / 24Pet | ~1,200 municipal partners, US + Canada footprint                    | Vendor-mediated     |

Canada has no national index. Coverage requires SMS vendor connectors (which already have Canadian footprints) plus direct provincial operator partnerships (BC SPCA, Ontario SPCA, Humane Canada network). Quebec requires bilingual handling and separate operator relationships. Canada is Wave 2.

**Behavioral trait extraction**: Shelter listing text must not be stored as flat booleans (`good_with_dogs: true`). The confirmed architecture uses an evidence-assertion model: every behavioral trait is stored with a value, calibrated confidence, evidence span (raw text), context (on-leash / in-home / kennel), certainty (observed / speculative / negated), source type (foster notes / staff notes / intake form / standardized test), and time scope. Marketing omission bias — shelters routinely exclude behavioral "stop signs" from public bios — means absence of mention cannot be treated as absence of behavior.

→ _Details in [`01-data-landscape-us-canada.md`](01-data-landscape-us-canada.md) and [`03-adoption-indexing-us-canada.md`](03-adoption-indexing-us-canada.md)._

### Pillar 3 — Human Profile Modeling

The v1 profile is a structured onboarding questionnaire producing a 5-component weighted fit score:

| Component               | What it captures                                            |
| ----------------------- | ----------------------------------------------------------- |
| Needs coverage          | Exercise/enrichment capacity vs dog energy and excitability |
| Risk alignment          | Training experience/willingness vs behavioral uncertainty   |
| Household compatibility | Children/pets/visitors vs dog sociability profile           |
| Care complexity fit     | Grooming/medical budget and time vs breed/dog complexity    |
| Bonding style           | Affectionate vs activity-oriented engagement preference     |

Hard constraints (allergies, housing rules, other pets, hours alone) are gate conditions applied before scoring, not score components.

The event logging schema (17 event types including `return_reported` with a reason taxonomy) must be instrumented from day one — even in v1 — so that v2 ML training data accumulates from first user. Every impression must log `position` to enable inverse propensity scoring for position-bias correction in the v2 ranker.

→ _Details in [`08-human-profile-modeling-research.md`](08-human-profile-modeling-research.md)._

---

## Platform Architecture

The platform is a 5-layer clearinghouse. Each layer produces a clean handoff to the next:

```
Layer 1  SOURCE ACQUISITION
         Connector types: api_connector | feed_connector | selenium_connector | manual_curation_connector
         Per-run artifacts: run manifest + WARC/HTML snapshot + screenshots + HAR log

Layer 2  NORMALIZATION + IDENTITY RESOLUTION
         Field mapping → canonical schema
         Entity resolution: BreedIdentity, OrganizationIdentity, ListingIdentity, UserProfileIdentity
         Duplicate collapse: deterministic + probabilistic matching

Layer 3  DATA QUALITY + PROVENANCE
         Per-field provenance (source + URL + timestamp + confidence + license status)
         TraitAssertion contradiction detection (two-tier: ontology rules + NLI)

Layer 4  INTELLIGENCE SERVICES
         NLP trait extraction pipeline
         Breed-risk and fit scoring
         Human profile embeddings and preference models
         Matching and re-ranking

Layer 5  API CLEARINGHOUSE SURFACE
         Internal product API + future third-party partner endpoints
```

Connector tier model:

| Tier | Type            | Examples                               |
| ---- | --------------- | -------------------------------------- |
| A    | Official API    | RescueGroups, ShelterLuv, ShelterBuddy |
| B    | Partner feed    | FTP/SFTP export, XML/JSON syndication  |
| C    | Selenium scrape | Dynamic sites with no viable API       |
| D    | Manual curation | CAPTCHA-blocked or one-off sources     |

Selenium at scale: Kubernetes HPA workers + Selenium Grid (Router → New Session Queue → Distributor → Nodes) + at-least-once queue with DLQ isolation + `ExtractionSpec` versioning (treat selector changes as versioned releases, canary before rollout) + 4-layer breakage detection (unit tests → canary runs → synthetic monitoring → production alerts).

→ _Details in [`07-clearinghouse-api-architecture.md`](07-clearinghouse-api-architecture.md)._

---

## Key Design Principles

These cross-cut all three pillars and must be respected at the schema and API level:

1. **Evidence over labels.** Never store a single behavioral label. Store `value + confidence + evidence_spans + context + certainty + source`. The match layer reasons over evidence.

2. **Provenance on every field.** Every ingested field carries: source name, source URL, ingest timestamp, license status (`allowed` / `restricted` / `unknown`). Required for legal defensibility and trust scoring.

3. **Canonical identity separate from source listing.** A `Dog` entity is canonical and persistent. A `DogListing` is source- and time-bound. The same dog can appear across multiple sources. Deduplication happens at the canonical layer.

4. **Behavior traits expire.** A kennel intake assessment is not the same as a foster observation 4 weeks later. Every `TraitAssertion` carries `time_scope` and may carry `expires_at`. Never serve stale behavioral data as current.

5. **Contradictions are information, not noise.** When two sources contradict each other on the same dog and attribute, that conflict is stored and classified (`contextual_difference` / `soft_conflict` / `explicit_contradiction`) rather than collapsed. Unresolved explicit contradictions route to human review.

6. **Position bias is a product-level risk.** Ranking order determines which dogs accumulate interaction data. Without correction, the ranker will reinforce initial placement decisions. Every impression must log `position`; every ranker must apply inverse propensity scoring.

7. **API-first ingestion policy.** Official API > partner feed > scraping. AKC and OFA prohibit bulk extraction by terms. Do not scrape sources that prohibit it without explicit authorization.

→ _Details in [`04-data-model-proposal.md`](04-data-model-proposal.md) and [`05-access-legal-scraping-guidelines.md`](05-access-legal-scraping-guidelines.md)._

---

## Confirmed Decisions (from deep research)

| #   | Decision                                                                                                                                                                                                                       | Brief |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ----- |
| 1   | RescueGroups v5 is the MVP ingestion spine. Petfinder decommissioned 2025-12-02. Adopt-a-Pet is partner-gated — pursue as BD track.                                                                                            | 1     |
| 2   | Shelter management systems (not aggregator platforms) are the real source of truth. Build SMS connectors in order: RescueGroups → ShelterLuv → ShelterBuddy → PetPoint.                                                        | 2     |
| 3   | Canada has no national index. Use SMS vendor connectors + direct provincial operator partnerships. Wave 2 effort, not MVP.                                                                                                     | 3     |
| 4   | Breeder quality scoring is feasible via a 6-dimension framework anchored to CHIC/OFA verification. Display Quality Score and Evidence Confidence as separate signals. Never binary-label breeders.                             | 4     |
| 5   | Behavioral traits from shelter text must use an evidence-assertion model with 6-dimension ontology, per-assertion calibrated confidence, and two-tier contradiction detection. Flat booleans are architecturally insufficient. | 5     |
| 6   | Human profile model has a concrete v1 (rules + constraint gates) → v2 (ML ranker with bias correction) → v3 (adaptive elicitation + bandit) build path. Event schema must be instrumented from day one.                        | 6     |
| 7   | Selenium at scale requires: Kubernetes HPA, Selenium Grid, at-least-once queue + DLQ, ExtractionSpec versioning, 4-layer breakage detection, per-run artifact archiving.                                                       | 7     |

---

## What changed from pre-research assumptions

| Prior assumption                                            | Confirmed reality                                                                           |
| ----------------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| Petfinder API is available                                  | Decommissioned 2025-12-02. Not a data source.                                               |
| Adopt-a-Pet is a viable MVP API source                      | Partner-gated. Requires business relationship.                                              |
| Canada has a national adoption index                        | Does not exist. Must build from vendor connectors + partnerships.                           |
| Shelter listing platforms are the authoritative source      | They are downstream syndication. Shelter management systems are the source of truth.        |
| Breeder quality is hard to score objectively                | Feasible with 6-dimension framework anchored to CHIC/OFA registry verification.             |
| Human profiling can be added post-launch                    | Event logging schema must be designed from day one or v2 training data is permanently lost. |
| Flat behavioral fields like `good_with_dogs` are sufficient | Unsafe. Absence of mention ≠ absence of behavior. Evidence-assertion model required.        |

---

## Remaining Open Questions

From Brief 1 (adoption API):

- What commercial terms does Adopt-a-Pet offer partners? What is the typical onboarding timeline?
- Are any Petfinder commercial data deals still active or being negotiated by larger players?

From Brief 2 (shelter software):

- What are the exact API field names and rate limits for ShelterBuddy production access?
- Does ShelterLuv's partner API key include all behavioral attribute fields or only fixed attributes?

From Brief 3 (Canada):

- What is the Quebec CQLPA/privacy regulatory posture for cross-province listing data aggregation?
- Is Les Pattes Jaunes open to a data partnership or exclusive to its own platform?

From Brief 4 (breeder signals):

- What does OFA charge for licensed data reuse in a commercial recommendation product?
- Can CHIC numbers be looked up via a public machine-readable endpoint or only via web search?

From Brief 6 (human profiling):

- What return-rate benchmark (by reason category) should we set as a "good" outcome metric?
- What minimum number of interactions is needed before an ML ranker outperforms the rules baseline?

From Brief 7 (Selenium architecture):

- Which specific sources require Selenium vs having stable hidden JSON/GraphQL endpoints (discoverable via HAR)?
- What browser fingerprinting or bot detection are we likely to encounter at scale?

---

## Immediate Build Priorities (next 4 weeks)

| Week | Priority                                                                                                      | Output                                         |
| ---- | ------------------------------------------------------------------------------------------------------------- | ---------------------------------------------- |
| 1    | Define canonical data schema (Dog, Listing, Breed, Organization, Trait, InteractionEvent, ReturnReason)       | Schema doc + ER diagram                        |
| 1    | Build RescueGroups v5 connector (API key auth, search, pagination, behavior field mapping)                    | Working connector + field mapping table        |
| 2    | Design event logging schema (17 event types from Brief 6, including position field)                           | Event schema doc + instrumentation spec        |
| 2    | Stand up Selenium Grid + Kubernetes worker skeleton                                                           | Running infrastructure + health check endpoint |
| 3    | Build ShelterLuv connector (API key, fixed behavioral attributes, narrative fields)                           | Working connector                              |
| 3    | Build v1 constraint gate model (hard rule filtering for allergies, pets, hours alone, housing)                | Working filter stage                           |
| 4    | Build ShelterBuddy connector (incremental sync via `UpdatedSinceUtc`, behavior assessments)                   | Working connector                              |
| 4    | Run field-mapping audit: RescueGroups + ShelterLuv + ShelterBuddy temperament fields → canonical Trait schema | Field coverage matrix                          |

---

## Research Runs Status

| Brief                                      | File                                                              | Status   |
| ------------------------------------------ | ----------------------------------------------------------------- | -------- |
| Brief 1 — Adoption API Feasibility         | `research-runs/2026-03-01-brief-1-adoption-api-feasibility.md`    | accepted |
| Brief 2 — Shelter Software Integration Map | `research-runs/2026-03-01-brief-2-shelter-software-map.md`        | accepted |
| Brief 3 — Canada Index Landscape           | `research-runs/2026-03-01-brief-3-canada-index-landscape.md`      | accepted |
| Brief 4 — Breeder Quality Signals          | `research-runs/2026-03-01-brief-4-breeder-quality-signals.md`     | accepted |
| Brief 5 — Temperament NLP Extraction       | `research-runs/2026-03-01-brief-5-temperament-nlp.md`             | accepted |
| Brief 6 — Human Profile Modeling           | `research-runs/2026-03-01-brief-6-human-profile-modeling.md`      | accepted |
| Brief 7 — Selenium Scale Architecture      | `research-runs/2026-03-01-brief-7-selenium-scale-architecture.md` | accepted |

---

## Document Map

| Doc                                          | What it covers                                                                                             |
| -------------------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| `01-data-landscape-us-canada.md`             | Executive landscape: data availability by category, connector priority, Canada strategy, NLP extraction    |
| `02-source-catalog.md`                       | Source-by-source inventory with confirmed access status, auth method, rate limits, and field notes         |
| `03-adoption-indexing-us-canada.md`          | Deep dive on national vs regional adoption indexing; multi-source ingestion and deduplication requirements |
| `04-data-model-proposal.md`                  | Canonical schema: all entities including `TraitAssertion`, `TraitContradiction`, `DogListing`, `Breed`     |
| `05-access-legal-scraping-guidelines.md`     | Ingestion policy, AKC/OFA licensing risk flags, per-source legal posture, provenance requirements          |
| `06-research-backlog-and-ai-agent-briefs.md` | Research backlog, deep research prompts, completed run tracking, open questions                            |
| `07-clearinghouse-api-architecture.md`       | Full 5-layer clearinghouse architecture, connector tier model, Selenium scale design, golden signal KPIs   |
| `08-human-profile-modeling-research.md`      | Third pillar deep dive: v1–v3 model roadmap, 17-event schema, bias/privacy risks, annotation plan          |
| `research-runs/`                             | Raw output files from all 7 external deep research sessions                                                |
