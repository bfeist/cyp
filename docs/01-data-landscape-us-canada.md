# Data Landscape Summary (US + Canada)

_Last updated: 2026-03-01_

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

- Petfinder: large listing volume and individual pet fast-fact attributes visible in search cards.
- RescueGroups: explicit API/feeds and broad multi-site export model for partner orgs.
- 24Petconnect and PetPlace ecosystem: listing/search and lost/found pathways.
- Shelter Animals Count: national-level analytics and reporting dataset across US+Canada, but not a consumer adoption marketplace itself.
- Humane Canada: federation/accreditation role in Canada rather than a single nationwide live adoptable-pet index.

Conclusion: treat "national" as **aggregated coverage through private/federated networks**, not a single authoritative state-run source.

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

## Strategic Recommendation

For a rapid but defensible prototype:

- Use **API-first breed data** (e.g., API Ninjas + curated registry references).
- Use **adoption aggregator partnerships/APIs** where available (RescueGroups is a strong candidate).
- Use **Petfinder/marketplace pages only as discovery for partnership targets**, not bulk scraping by default.
- Build canonical schema + provenance from day one so each feature knows its confidence/licensing source.

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
