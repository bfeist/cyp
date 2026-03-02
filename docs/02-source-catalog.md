# Source Catalog (US + Canada)

_Last updated: 2026-03-01_

## Legend

- **Role**: Breed | Adoption | Breeder | Health | Sector analytics
- **Access**: API | Web listings | Report download | Needs partnership
- **Prototype fit**: High / Medium / Low
- **Caution**: legal/quality caveats

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
- Access: Web listings (developer docs endpoint not confirmed via automated fetch)
- Data observed: age, sex, breed/mix, shelter name, compatibility flags (`good with`), house-trained indicator, location
- Prototype fit: High (discovery), Medium (ingestion pending approved API/terms path)
- Caution: verify current API program + terms before any systematic collection.

### 6) RescueGroups

- URLs:
  - https://www.rescuegroups.org/
  - https://www.rescuegroups.org/services/adoptable-pet-data-api/
- Role: Adoption
- Access: API + platform tools
- Data observed: API key flow, rich animal/org fields, long-standing adoption data tooling
- Prototype fit: Very High
- Caution: implement per API terms and org-level permissions.

### 7) Shelterluv

- URL: https://www.shelterluv.com/
- Role: Shelter operations platform
- Access: SaaS platform (no public open listing feed observed)
- Data observed: shelter workflow tooling, donations, operations; terms mention restrictions on automated high-volume access
- Prototype fit: Medium (integration partner candidate)
- Caution: likely partnership/integration required.

### 8) 24Petconnect / PetPlace bridge

- URLs:
  - https://www.24petconnect.com/
  - https://www.petplace.com/pet-adoption/
- Role: Adoption + lost/found ecosystem
- Access: Web listings
- Data observed: “search adoptable pets all in one place” positioning, integration with lost/found ecosystem
- Prototype fit: Medium
- Caution: terms PDF could not be parsed fully via automated fetch; manual review required.

---

## Breeder / Purchase Sources

### 9) AKC Marketplace

- URL: https://marketplace.akc.org/puppies
- Role: Breeder/purchase
- Access: Web listings
- Data observed: large listing counts, breeder names, locations, puppy attributes
- Prototype fit: High (reference), Low/Medium (ingestion depends on permission)
- Caution: governed by AKC terms/restrictions.

### 10) CKC Puppy List (Canada)

- URL: https://www.ckc.ca/en/Choosing-a-Dog/PuppyList
- Role: Breeder/purchase
- Access: Searchable web directory
- Data observed: searchable breeder contact listings by breed
- Prototype fit: High for Canada landscape mapping
- Caution: verify allowed usage and contact-data compliance.

### 11) Good Dog

- URL: https://www.gooddog.com/
- Role: Breeder + shelter/rescue marketplace
- Access: Web platform
- Data observed: vetted-network claims, breed pages, breeder/shelter directory concepts
- Prototype fit: Medium-High
- Caution: likely contractual access needed for bulk integration.

---

## Health / Genetics / Risk Sources

### 12) OFA + CHIC

- URLs:
  - https://ofa.org/
  - https://ofa.org/chic-programs/
- Role: Health
- Access: Database/search and reports
- Data observed: breed-specific recommended health screenings, CHIC framework, large cumulative testing volume
- Prototype fit: Very High (for health-risk metadata)
- Caution: OFA copyright/reproduction limits explicitly stated; license/permission path required.

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
