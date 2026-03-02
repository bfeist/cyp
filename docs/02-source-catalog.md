# Source Catalog (US + Canada)

_Last updated: 2026-03-02 (updated with deep research run findings)_

> **Status legend updated from research runs:**
>
> - `✅ Confirmed active` — research confirmed current access
> - `❌ Decommissioned` — no longer available
> - `🔒 Partner-gated` — requires business relationship to access
> - `⚠️ Vendor-mediated` — access requires vendor coordination
> - `🔍 Pending verification` — not yet confirmed by research

## Legend

- **Role**: Breed | Adoption | Shelter SMS | Breeder | Health | Sector analytics
- **Access**: API | Web listings | Report download | Needs partnership
- **Prototype fit**: High / Medium / Low

---

## Breed / Registry / Standards Sources

### 1) AKC Dog Breeds

- URL: https://www.akc.org/dog-breeds/
- Role: Breed
- Access: Web listings
- Data observed: breed pages, grouping, characteristic collections
- Prototype fit: High (reference), Low (bulk ingestion unless licensed)
- Caution: AKC Terms prohibit scraping/automated extraction for commercial usage.

### 2) CKC Choosing a Breed (Canada)

- URL: https://www.ckc.ca/en/Choosing-a-Dog/Choosing-a-Breed
- Role: Breed
- Access: Web listings
- Data observed: group taxonomy and breed-level decision dimensions (size, coat, energy, purpose, temperament, allergies)
- Prototype fit: High (Canada reference)
- Caution: use terms-compliant ingestion path.

### 3) UKC Breeds / Breed Standards

- URL: https://www.ukcdogs.com/breeds
- Role: Breed + Breeder entry point
- Access: Web + related standards pages
- Data observed: breed standards and breeder search pathways
- Prototype fit: Medium-High
- Caution: confirm reuse permissions before extraction.

### 4) FCI Nomenclature

- URL: https://www.fci.be/en/nomenclature/
- Role: Breed taxonomy/global standards reference
- Access: Web
- Data observed: breed groups, definitive/provisional recognition, standards-related links
- Prototype fit: Medium-High
- Caution: international standards and naming normalization complexity.

---

## Adoption / Shelter Listing Sources

### 5) Petfinder dog search (US/CA)

- URLs:
  - https://www.petfinder.com/search/dogs-for-adoption/us/
  - https://www.petfinder.com/search/dogs-for-adoption/ca/
- Role: Adoption
- **Status: ❌ Decommissioned (public API ended 2025-12-02). Do not use as a data source.**
- Supported replacement: Custom Pet List embeddable widget and deep links only
- Fallback path: use widget for web surfaces where embedding is acceptable; deep-link to Petfinder pet detail pages for mobile
- Data previously observed: age, sex, breed/mix, shelter name, compatibility flags, house-trained, location
- Prototype fit: **None for data ingestion. Medium for outbound traffic destination.**
- Confirmed by: Brief 1

### 6) RescueGroups

- URLs:
  - https://www.rescuegroups.org/
  - https://rescuegroups.org/services/adoptable-pet-data-api/
  - https://api.rescuegroups.org/v5/public/docs (v5 API docs)
- Role: Adoption + Shelter SMS hub
- **Status: ✅ Confirmed active — v5 REST API publicly documented and operational**
- Auth: `Authorization: <API_KEY>` header for public endpoints; `Content-Type: application/vnd.api+json`
- Rate limits: no published quota; returns 429 on abnormal load; max 250 per request
- Coverage: 6,200+ partner orgs; exports to 200+ adoption websites/services
- Behavioral fields confirmed: structured behavior/environment fields (energy level, exercise needs, housetrained, reaction to new people, OK with cats/dogs/kids, affectionate, timid/skittish)
- Canadian presence: yes (rescues using RG as syndication hub)
- Prototype fit: **Very High — MVP primary source**
- Constraints: temporary use/display data rights; Pet Adoption Tracker image required on public pet detail pages; org-level opt-out; deletion on termination
- Confirmed by: Briefs 1 + 2 + 3

### 7) ShelterLuv

- URL: https://www.shelterluv.com/
- Role: Shelter SMS (source of truth for many orgs)
- **Status: ✅ Confirmed active API — API key issued via Jotform request**
- Auth: account-scoped API key
- Coverage: all 50 US states; confirmed Canadian SPCA usage (Saskatoon SPCA, others)
- Behavioral fields confirmed: Fixed Behavioral Attributes + Custom Behavioral Attributes; fixed attributes directly aligned to Petfinder/Adopt-a-Pet upload specs; per-attribute text notes + narrative listing description
- Prototype fit: **Very High — Connector Priority #2**
- Canadian presence: yes (Saskatoon SPCA cites real-time ShelterLuv updates)
- Confirmed by: Briefs 2 + 3

### 8) 24Petconnect / PetPoint / Petango

- URLs:
  - https://24petconnect.com/
  - https://www.24pet.com/products/petpoint
  - https://ws.petango.com/webservices/wsadoption.asmx (legacy SOAP endpoint)
- Role: Shelter SMS + adoption listing
- **Status: ⚠️ Vendor-mediated — requires vendor coordination for API access**
- Coverage: ~1,200 shelter partners (Best Friends data); confirmed municipal usage in Canada (City of St. John’s NL, Ontario SPCA, and others run PetPoint-branded adoptable pages)
- Behavioral fields: structured compatibility flags (Good with Cats/Dogs/Kids, Housetrained) + Petango “Adoption Description Memo” for narrative
- Legacy SOAP: `AdoptableSearch` + `AdoptableDetails` operations exist; country code prefix (US/CA) visible in PetPoint IDs
- Prototype fit: **High — Connector Priority #4. High coverage leverage across North American municipalities.**
- Confirmed by: Briefs 2 + 3

### 8a) ShelterBuddy _(new entry from research)_

- URLs:
  - https://shelterbuddy.atlassian.net/wiki/spaces/SbApi/overview
- Role: Shelter SMS
- **Status: ✅ Confirmed active API — public Swagger-style docs, incremental sync pattern**
- Auth: API key
- Key feature: `UpdatedSinceUtc` parameter for incremental polling; explicit behavior assessment endpoints with `Notes` free-text field
- Coverage: 200+ shelters; confirmed Canadian humane society usage (Regina Humane Society on ShelterBuddy-hosted portal)
- Prototype fit: **Very High — Connector Priority #3**
- Confirmed by: Briefs 2 + 3

### 8b) Adopt-a-Pet

- URLs:
  - https://www.adoptapet.com/
  - https://partner-apis.adoptapet.com/
- Role: Adoption aggregator
- **Status: 🔒 Partner-gated. API not available to non-partners/students. Must email to apply for partnership.**
- Legacy API pattern: HTTP GET + querystring (`key=`, `v=`), endpoints: `/search/pet_details`, `/search/pets_at_shelter`, `/search/pets_at_shelters`. Max `end_number` up to 25,000; `pet_search` max 10,000 with 500/page pagination
- Error on rate breach: `503 resource_unavailable`
- Prototype fit: **None for MVP. Pursue as formal BD track for Phase 2.**
- Confirmed by: Brief 1

---

## Breeder / Purchase Sources

### 9) AKC Marketplace

- URL: https://marketplace.akc.org/puppies
- Role: Breeder/purchase
- Access: Web listings
- **Status: 🔍 Direct field inspection blocked during research. Breeder program pages accessible.**
- Quality signals available via program badges: Bred with H.E.A.R.T. (health testing + education + accountability + responsible owner + tradition of sport); Breeder of Merit (event involvement, AKC club membership, 100% puppy registration, health certifications)
- Data observed: program-level evidence of commitments; direct listing fields not inspectable
- Prototype fit: High (reference), Low (bulk ingestion)
- Confirmed by: Brief 4

### 10) CKC Puppy List (Canada)

- URL: https://www.ckc.ca/en/Choosing-a-Dog/PuppyList
- Role: Breeder/purchase
- Access: Searchable web directory
- Data fields confirmed: name, breed, contact mechanism; enhanced listings add phone, website, ad copy, photos
- **No standardized welfare fields** (health testing, contracts, socialization) in listing data. Buyers must ask breeder directly.
- CKC breeders must adhere to CKC Code of Ethics/Practice; check CKC Disciplined Persons List for removal flags.
- Prototype fit: **High for Canada landscape mapping / directory pointer. Low for quality scoring input.**
- Confirmed by: Brief 4

### 11) Good Dog

- URL: https://www.gooddog.com/
- Role: Breeder + shelter/rescue marketplace
- Access: Web platform
- **Most structured breeder quality data of any listing source confirmed.**
- Fields confirmed: health testing levels (Good/Great/Excellent), socialization screening, buyer-protection policies, lifetime take-back commitment; all based on self-reported practices
- Limitation: Good Dog does not verify health testing as part of screening; confidence must be stored separately
- Prototype fit: **High — use as quality signal benchmark. Treat as directory pointer — collect data directly from breeders for scoring.**
- Confirmed by: Brief 4

---

## Health / Genetics / Risk Sources

### 12) OFA + CHIC

- URLs:
  - https://ofa.org/
  - https://ofa.org/chic-programs/
- Role: Health
- Access: Database/search and reports
- **CHIC certification confirmed as Tier 3 verification** (strongest evidence tier): requires breed-recommended screenings to be completed AND results publicly disclosed. CHIC number ≠ "all normal" — it means “tests done and disclosed.”
- OFA normal results are public; abnormal result publication may depend on test type and owner consent unless CHIC requires disclosure
- OFA explicitly copyrights database content; public reproduction requires express permission
- CHIC/OFA link-out strategy: store identifier + URL, do not republish full registry records
- Prototype fit: Very High (for health-risk metadata + Tier 3 breeder score verification)
- Confirmed by: Brief 4

---

## Open/Commercial Structured APIs

### 13) API Ninjas Dogs API

- URL: https://www.api-ninjas.com/api/dogs
- Role: Breed structured features
- Access: API
- Data observed: rich numeric trait fields + size/lifespan + social compatibility and behavior scores
- Prototype fit: Very High for quick prototyping
- Caution: commercial terms, request limits, and model provenance review needed.

### 14) TheDogAPI

- URL: https://thedogapi.com/
- Role: Breed/health/intelligence platform
- Access: Product/API offerings
- Data observed: enterprise positioning around medical/behavioral/breed data
- Prototype fit: Medium-High
- Caution: commercial pricing and data rights negotiation likely required.

### 15) Dog CEO API

- URL: https://dog.ceo/dog-api/
- Role: Breed image/media
- Access: Open image API
- Data observed: random and breed image endpoints
- Prototype fit: Medium (UI/media enrichment only)
- Caution: not a deep behavioral/health data source.

---

## Sector Analytics / Participation Coverage

### 16) Shelter Animals Count

- URL: https://www.shelteranimalscount.org/
- Role: Shelter analytics and national trend reporting (US + Canada)
- Access: dashboards, reports, data-use agreement workflows
- Data observed: large participating-organization network and annual reporting
- Prototype fit: High (market context + benchmarking), Medium (direct listing feed)
- Caution: not a direct “adoptable now” consumer listing API.

### 17) Humane Canada

- URLs:
  - https://www.humanecanada.ca/
  - https://humanecanada.ca/en/sector-professionals/humane-canada-accreditation
- Role: Federation/accreditation (Canada)
- Access: reports, standards, member ecosystem
- Data observed: national federation and accreditation framework
- Prototype fit: Medium (network mapping and partnerships)
- Caution: not a single canonical live adoption index.
