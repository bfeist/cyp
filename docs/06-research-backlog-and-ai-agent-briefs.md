# Iterative Research Backlog + External AI Session Briefs

_Last updated: 2026-03-01_

## Quick answer to "all at once or one at a time?"

- If you are running **solo in one ChatGPT account**: run **one at a time** in the order below.
- If you can run **multiple deep-research sessions in parallel** (multiple tabs/accounts/tools): run in **waves**.

Recommended wave plan:

- **Wave 1 (parallel):** Briefs 1, 2, 6
- **Wave 2 (parallel):** Briefs 3, 4, 7
- **Wave 3 (parallel):** Brief 5 + any follow-up brief from gaps found in Waves 1–2

This gives you speed without losing control of outputs.

## Exactly what to do each session

Before each deep research run:

1. Start a brand-new chat/session.
2. Paste the corresponding brief prompt from this file.
3. Add this final line:
   - `Return output as markdown with headings exactly matching the requested format.`
4. After the report returns, save it to:
   - `docs/research-runs/YYYY-MM-DD-<brief-name>.md`

Suggested file names:

- `docs/research-runs/2026-03-01-brief-1-adoption-api-feasibility.md`
- `docs/research-runs/2026-03-01-brief-2-shelter-software-map.md`
- `docs/research-runs/2026-03-01-brief-3-canada-index-landscape.md`
- `docs/research-runs/2026-03-01-brief-4-breeder-quality-signals.md`
- `docs/research-runs/2026-03-01-brief-5-temperament-nlp.md`
- `docs/research-runs/2026-03-01-brief-6-human-profile-modeling.md`
- `docs/research-runs/2026-03-01-brief-7-selenium-scale-architecture.md`

---

## Priority Backlog (next 2-4 weeks)

## P0 (must clarify)

1. Confirm active Petfinder API program (current docs, auth flow, terms, quota, allowed use).
2. Confirm Adopt-a-Pet data access options (API/partnership/export) and licensing terms.
3. Identify the top 10 shelter-management platforms by market share in US/Canada and their public integration options.
4. Validate legal permissibility and pricing for OFA/CHIC data reuse in a commercial matching application.
5. Define the v1 clearinghouse API contract that unifies breed, listing, and human profile data.
6. Define human-trait ontology and profile signal model (explicit + implicit).

## P1 (high value)

7. Build standardized “temperament field dictionary” across adoption channels.
8. Build standardized “breed trait ontology” mapping AKC/CKC/FCI/API-Ninjas naming differences.
9. Identify Canadian provincial shelter networks with syndication feeds.
10. Identify breeder-directory anti-puppy-mill compliance signals that can be modeled.
11. Design Selenium extraction framework and source tiering policy for dynamic sites.
12. Define event instrumentation for behavioral preference inference.

## P2 (enhancement)

13. Collect representative sample of real listing descriptions to test NLP extraction quality.
14. Design confidence scoring model for individual temperament claims.
15. Evaluate image-based breed inference APIs for mixed breeds.
16. Run explainability UX experiments for match trust and profile correction.

---

## Run Order (solo execution)

Run these in this order if you are doing one-by-one:

1. Brief 6 — Human Profile Modeling Beyond Questionnaires
2. Brief 2 — US + Canada Shelter Software Integration Map
3. Brief 7 — Selenium-at-Scale Acquisition Architecture
4. Brief 1 — Adoption API Feasibility
5. Brief 3 — Canada-specific Adoption Index Landscape
6. Brief 4 — Breeder Data Quality + Ethics Signals
7. Brief 5 — Temperament Extraction from Listing Text

Why this order:

- Start with user-profile strategy and ingestion architecture (core product differentiator).
- Then validate source ecosystem and channel specifics.
- End with extraction/modeling details that depend on the earlier decisions.

---

## External AI Deep Research Briefs (copy/paste)

## Brief 1 — Petfinder/Adoption API Feasibility

**Objective:** verify current programmatic access options for adoption data at production quality.

Prompt:

"You are conducting technical due diligence for a US/Canada pet-adoption matching app.

Tasks:

1. Determine current Petfinder developer/API availability as of today.
2. Provide official docs links, auth flow, rate limits, and key endpoint examples.
3. Compare with RescueGroups and Adopt-a-Pet access models.
4. Provide the best ingestion strategy for MVP and production.
5. Identify likely blockers and fallback ingestion paths.

Output format:

- Executive summary
- Source-by-source comparison table
- Endpoint and auth summary by source
- Blockers and fallback options
- Final recommendation"

## Brief 2 — US + Canada Shelter Software Integration Map

**Objective:** find where the data actually originates operationally.

Prompt:

"Map the shelter/rescue software ecosystem in the US and Canada for 2026.

Tasks:

1. Identify top platforms used by shelters/rescues (e.g., Shelterluv, PetPoint, Chameleon, RescueGroups, others).
2. For each platform, identify whether public APIs, export feeds, partner integrations, or listing syndication exist.
3. Note whether temperament/behavior fields are structured or free-text.
4. Assess feasibility of building one connector per platform.

Output format:

- Ranked platform list
- Integration capability matrix
- Field availability matrix
- Recommended integration order for an MVP"

## Brief 3 — Canada-specific Adoption Index Landscape

**Objective:** deepen Canada coverage where data is often fragmented.

Prompt:

"Research Canada-specific adoption listing infrastructure.

Tasks:

1. Determine whether any national Canadian adoptable-pet index exists with broad coverage.
2. Identify provincial/regional shelter networks and major aggregators.
3. Identify data-sharing standards, if any, across Canadian humane societies/SPCAs.
4. Provide the best path to obtain broad Canadian listing coverage.

Output format:

- National vs provincial landscape summary
- Province-by-province key channels
- Partnership targets with rationale
- Risks and data gaps"

## Brief 4 — Breeder Data Quality + Ethics Signals

**Objective:** support responsible purchase matching while avoiding low-quality breeders.

Prompt:

"Design a breeder-quality signal framework for a dog matching platform.

Tasks:

1. Identify measurable breeder quality indicators (health testing evidence, registrations, CHIC/OFA references, contracts, socialization practices).
2. Compare data availability across AKC Marketplace, CKC Puppy List, UKC, Good Dog, and independent sites.
3. Propose a scoring rubric with transparent explanations.
4. Identify practical risks in ranking or labeling breeders and mitigation options.

Output format:

- Indicator framework
- Data availability by source
- Scoring rubric draft (0-100)
- Risk/mitigation section"

## Brief 5 — Temperament Extraction from Listing Text

**Objective:** extract individual-dog behavior with confidence scores.

Prompt:

"Design an NLP extraction approach for shelter/rescue dog descriptions.

Tasks:

1. Propose ontology for temperament and behavior labels (reactivity, sociability, prey drive, energy, separation behavior, trainability cues).
2. Define extraction pipeline from noisy descriptions to structured attributes.
3. Include uncertainty/confidence scoring and contradiction detection.
4. Provide annotation guidelines and evaluation metrics.

Output format:

- Ontology draft
- Pipeline architecture
- Example input/output pairs
- Evaluation plan"

## Brief 6 — Human Profile Modeling Beyond Questionnaires

**Objective:** design a modern, adaptive profile system similar to advanced dating/recommendation apps.

Prompt:

"Design a human-profile modeling framework for a dog-owner matchmaking application.

Tasks:

1. Define profile dimensions that predict successful owner↔dog outcomes.
2. Compare explicit questionnaire signals vs implicit behavioral signals.
3. Propose adaptive preference elicitation methods (pairwise choices, conversational prompts, active learning).
4. Provide a modeling strategy (baseline rules + ML ranker + online feedback loop).
5. Specify explainability approach and profile correction UX.

Output format:

- Trait ontology and signal map
- Data model and event schema recommendations
- MVP vs phase-2 model roadmap
- Risks (bias/privacy) and mitigations"

## Brief 7 — Selenium-at-Scale Acquisition Architecture

**Objective:** establish a robust browser-automation ingestion framework for high-value dynamic sources.

Prompt:

"Design a production-grade Selenium data acquisition architecture for a multi-source pet data clearinghouse.

Tasks:

1. Define crawler orchestration and queue strategy for hundreds of source jobs/day.
2. Define selector management, breakage detection, and canary test strategy.
3. Propose source policy tiers (API/feed/scrape/manual) and promotion/demotion rules.
4. Specify audit/provenance artifacts to store for each extraction run.
5. Provide operational KPIs and incident playbooks.

Output format:

- Reference architecture diagram (text description)
- Component responsibilities
- Failure modes and mitigations
- MVP implementation plan"

---

## Required quality bar for each returned report

Before accepting a report as complete, confirm it includes all of the following:

- Concrete tables, not only narrative text
- Specific URLs for factual claims
- Explicit assumptions
- A clear “what to build next week” section
- At least 10 unresolved questions for follow-up

If any are missing, ask the same agent:

`Revise the report to include the missing sections and keep all original findings.`

---

## Consolidation workflow (when reports are done)

After all briefs are complete:

1. Create `docs/research-runs/MASTER-SUMMARY.md` containing:
   - decisions
   - open questions
   - final architecture changes
2. Update these files from the summary:
   - `docs/01-data-landscape-us-canada.md`
   - `docs/02-source-catalog.md`
   - `docs/07-clearinghouse-api-architecture.md`
   - `docs/08-human-profile-modeling-research.md`
3. Add one section at bottom of this file:
   - `## Completed Research Runs`
   - list each run file and status (`accepted`, `needs-follow-up`)

---

## Return-to-repo workflow

When you get a deep research report back:

1. Save it in `docs/research-runs/`.
2. Add a one-paragraph synthesis to `docs/01-data-landscape-us-canada.md`.
3. Update `docs/02-source-catalog.md` with any new source status.
4. Update `docs/07-clearinghouse-api-architecture.md` and `docs/08-human-profile-modeling-research.md` with any architecture/model decisions.
5. Track unresolved questions at the bottom of this file.

## Completed Research Runs

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

## Open Questions Log

**Resolved by research runs:**

- ~~Is there a currently active, public Petfinder API?~~ → **No. Decommissioned 2025-12-02. (Brief 1)**
- ~~Which breeder channels provide verifiable health-testing data?~~ → **CHIC/OFA = strongest. Good Dog = most structured self-reported. CKC/UKC = directory only. (Brief 4)**

**Still open:**

- What commercial terms does Adopt-a-Pet offer partners, and what is the typical onboarding timeline?
- Are any Petfinder commercial data deals still active with larger players?
- What are exact API field names and rate limits for ShelterBuddy production access?
- Does ShelterLuv's partner API key include all behavioral attribute fields or only fixed attributes?
- What is Quebec's CQLPA/privacy regulatory posture for cross-province listing data aggregation?
- Is Les Pattes Jaunes (Quebec) open to a data partnership?
- What does OFA charge for licensed data reuse in a commercial recommendation product?
- Can CHIC numbers be looked up via a public machine-readable endpoint or only via web search?
- What return-rate benchmark (by category) should we set as a baseline "good" outcome metric?
- What minimum number of interactions is needed before an ML ranker outperforms the rules baseline?
- Which specific sources require Selenium vs having stable hidden JSON/GraphQL endpoints discoverable via HAR?
- What bot detection/fingerprinting is likely to be encountered at scale on high-value dynamic sources?
