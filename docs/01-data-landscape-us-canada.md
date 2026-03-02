# Data Landscape Summary (US + Canada)

_Last updated: 2026-03-02 (updated with deep research run findings)_

## TL;DR — Updated After Research Runs

> Research runs (Briefs 1–4, 6–7) confirmed and materially updated several prior assumptions.
> See `00-executive-summary.md` for the full decision record.

**Critical changes from research:**

- **Petfinder API decommissioned** (2025-12-02). Not usable as a data source. Widget/deep-link only.
- **RescueGroups v5 REST API confirmed active** and is the primary MVP ingestion source.
- **Adopt-a-Pet API is partner-gated.** Cannot obtain a key without a business relationship. Queue for BD track.
- **Shelter management systems (SMS) are the actual data origin**, not Petfinder/Adopt-a-Pet. SMS connector priority: RescueGroups → ShelterLuv → ShelterBuddy → PetPoint.
- **Canada: no national index exists.** Coverage requires vendor connectors + direct provincial operator partnerships.
- **Breeder quality scoring is feasible** with a 6-dimension framework anchored to CHIC/OFA registry links.
- **Human profile event schema must be designed from day one** or v2 ML training data will not exist.

## TL;DR

- There is **no single mandatory government-run national adoption listing system** covering all shelters in either the US or Canada.
- The ecosystem is **federated/marketplace-driven**: shelters and rescues publish into platforms (Petfinder, RescueGroups, shelter software exports, 24Petconnect/PetPlace ecosystem, local shelter sites).
- For breed intelligence, strongest practical backbone is:
  - Registry standards (AKC, CKC, UKC, FCI)
  - Structured breed APIs (API Ninjas Dogs API, TheDogAPI offerings)
  - Health screening datasets (OFA + CHIC ecosystem)
- For individual adoptable-dog temperament data, Petfinder-style listings show fields such as:
  - `good with (dogs/cats/children)`
  - house-trained
  - age, sex, breed/mix, shelter, location

## What this means for your product concept

Your app can become the “matching layer” by unifying three separate data planes:

1. **Breed baseline profile** (stable, mostly static):
   - temperament tendencies, activity needs, grooming, trainability, size, lifespan, known health concerns
2. **Individual dog profile** (dynamic):
   - shelter/rehome availability, observed behavior, compatibility tags, constraints, geography
3. **Acquisition channel profile**:
   - adoption (shelter/rescue) and purchase (breeder) pathways with source trust metadata

Also add a fourth operational capability: a **clearinghouse API layer** that normalizes and serves all three planes via your own stable endpoints.

## Key Findings by Topic

## 1) Breed-level data availability

High-confidence sources exist with broad breed coverage:

- AKC breed directory and category pages
- CKC breed pages and filtering dimensions (size, coat type, energy, purpose, temperament)
- UKC breed standards and breeder directory entry points
- FCI nomenclature + standards and group taxonomies

Structured API options exist (commercial/open tiers):

- API Ninjas Dogs API with explicit feature fields (trainability, energy, barking, protectiveness, social compatibility, height/weight/lifespan)
- TheDogAPI enterprise product positioning around health/behavioral/breed data

Health-centric depth source:

- OFA + CHIC programs (breed-specific health screening protocols, longitudinal health-testing orientation)

## 2) Adoption indexing reality (US/Canada)

Evidence points to **decentralized listing** with national-scale aggregators:

- Petfinder: **API decommissioned 2025-12-02.** Treat as widget/deep-link destination only. (Confirmed by Brief 1.)
- RescueGroups: **Active v5 REST API confirmed.** API key auth, search/filter/paginate, 429 on overload. Primary MVP source. (Confirmed by Briefs 1 + 2.)
- Adopt-a-Pet: **Partner-gated API.** Exists but inaccessible without signed partnership. Pursue BD track. (Confirmed by Brief 1.)
- 24Petconnect/Petango: SOAP-style `wsAdoption` endpoint exists; visible in municipal Canadian deployments. Connector candidate for Phase 2.
- Shelter Animals Count: national analytics, not a live inventory. Expanding to Canada (pilot with Humane Canada).
- Humane Canada: federation role only. Not a pet-level listing database. (Confirmed by Brief 3.)
- Canada: **no national pet index exists.** Coverage path = vendor connectors + provincial operator partnerships. (Confirmed by Brief 3.)

Conclusion: treat "national" as **aggregated coverage through private/federated networks**, not a single authoritative state-run source. For Canada specifically, build connectors to the vendor stacks (ShelterBuddy, ShelterLuv, PetPoint/24Petconnect) with Canadian footprints, and sign direct agreements with top provincial operators.

### Shelter software connector priority (confirmed by Brief 2)

| Priority | Platform         | Evidence                                                                                               | Integration surface      |
| -------- | ---------------- | ------------------------------------------------------------------------------------------------------ | ------------------------ |
| 1        | RescueGroups     | 6,200+ partner orgs, v5 REST API documented and active                                                 | REST API + FTP download  |
| 2        | ShelterLuv       | 50-state US usage, Canadian SPCA usage, API key model, behavioral attributes aligned to listing specs  | API key + polling        |
| 3        | ShelterBuddy     | Incremental sync (`UpdatedSinceUtc`), behavior assessment endpoints, Swagger docs, Canadian HS usage   | REST API                 |
| 4        | PetPoint / 24Pet | ~1,200 shelter partners, municipal US + Canada deployments (PetPoint-branded pages visible in NL + ON) | Vendor-mediated / iFrame |

## 3) Breeder/purchase data availability

- US: AKC Marketplace has large searchable puppy listings with breeder profile/location metadata.
- Canada: CKC Puppy List provides searchable breeder contact listings by breed.
- UKC has breeder search entry points and standards ecosystem.
- Marketplace-style networks (e.g., Good Dog) aggregate breeders/shelters/rescues with screening claims.

## 4) Access feasibility: API vs scraping

Practical hierarchy:

1. Official API / official export / partnership feed
2. Licensed data pull from marketplaces/registries
3. Public HTML scraping only where terms permit and legal review approves

For this concept, large-scale scraping can be treated as an explicit ingestion tier (not a fallback), especially where dynamic pages require browser automation and no practical feed exists.

Recommended extraction split:

- API/feed connectors for stable high-volume pulls.
- Selenium connectors for JS-heavy dynamic content and interaction-driven extraction.
- Manual curation path for edge sources and corrections.

Observed hard constraints:

- AKC terms explicitly prohibit scraping/automated extraction for commercial use without separate authorization.
- AKC reproduction policy requires explicit permission for reuse of online materials.
- OFA content explicitly claims copyright and restricts public reproduction without permission.
- Several key pages (e.g., some Petfinder/Adopt-a-Pet developer paths) were inaccessible via automated fetch and need direct/manual verification.

## Strategic Recommendation (updated 2026-03-02)

- Use **API-first breed data** (e.g., API Ninjas + curated registry references).
- **RescueGroups v5 API is the confirmed MVP adoption data source.** Build this connector first.
- **Petfinder is not a data source.** Use only for widget embeds and outbound deep-links.
- **Adopt-a-Pet**: pursue formal partner onboarding. Do not block MVP on it.
- For Canada: start with the vendor connectors that have Canadian footprints (ShelterBuddy, ShelterLuv, PetPoint), then add provincial operator partnerships.
- Build canonical schema + provenance from day one so each feature knows its confidence/licensing source.
- **Log interaction event positions from day one** (required for unbiased v2 ML training).

## 5) Third pillar gap: Human profile intelligence

Current landscape research confirms ample dog/breed/listing data, but the hardest differentiator is high-fidelity **human profile modeling**.

Required direction:

- move beyond questionnaire-only profiling
- add behavioral and preference-inference signals
- create transparent, revisable profile state
- tie profile quality to real outcomes (successful match retention vs mismatch/return)

Research and architecture details are captured in:

- `07-clearinghouse-api-architecture.md`
- `08-human-profile-modeling-research.md`

## 6) Individual-dog behavioral description quality (NLP extraction track)

_(Confirmed by Brief 5)_

Shelter and rescue listing descriptions are the only widely available per-dog behavioral signal. Quality varies widely (foster notes are most reliable; public bios are lowest-fidelity marketing copy), but NLP extraction is confirmed feasible.

**Key points for product and data architecture:**

- Flat boolean fields (`good_with_dogs`, `good_with_cats`) drawn directly from shelter submissions are unreliable and architecturally too coarse. Replace with evidence-assertion objects: `value + confidence + evidence_spans + context + certainty + source_type + time_scope`.
- Marketing omission bias is material: shelters routinely omit behavioral "stop signs" from public copy. Fetch full text from all source types (intake form, foster notes, behavior assessment) when connector access allows.
- Context is as important as polarity: a dog reactive on-leash may be fine off-leash. Never collapse context into a single label.
- Contradictions between sources are common and structurally meaningful; store and classify them rather than resolving silently.
- Annotation effort (3,000–8,000 labeled listings) should begin as RescueGroups ingestion is being built — they are parallel tracks.

Schema: see `04-data-model-proposal.md` `TraitAssertion` and `TraitContradiction` entities.
