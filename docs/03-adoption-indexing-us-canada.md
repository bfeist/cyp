# Adoption Indexing in the US and Canada

_Last updated: 2026-03-01_

## Core Question

Is there a national adoption indexing system, or is everything regional?

## Short Answer

- **Operationally regional and organization-driven** at intake and care level.
- **National-scale visibility** happens through private aggregators/platforms, not a single mandatory government index.

## Why this matters for your product

Your app should not assume one authoritative national feed. Instead, design for:

- multi-source ingestion
- duplicate detection across aggregators
- per-record provenance
- source-level trust and freshness scoring

## Observed Ecosystem Layers

## Layer A: Origin systems (where pet records are created)

- Municipal shelters
- Private humane societies/SPCAs
- Foster-based rescues
- Rescue collectives
- Shelter software platforms (e.g., Shelterluv-type systems)

This layer is decentralized and operationally local/regional.

## Layer B: Aggregators / distribution channels

- Petfinder listings (very large public index footprint)
- RescueGroups export/API ecosystem
- 24Petconnect / PetPlace style networks
- Direct shelter websites and social channels

This layer creates the user-facing perception of national scale.

## Layer C: Analytics-only national reporting

- Shelter Animals Count functions as a centralized dataset/reporting initiative for trends, participation dashboards, and annual reports.
- It is not equivalent to a consumer adoption marketplace with real-time listing semantics.

## US vs Canada Interpretation

## United States

- Strong aggregator presence and high adoption-listing discoverability through private platforms.
- No evidence in this research pass of a single federally mandated national listing registry for all shelters.

## Canada

- Humane Canada functions as a national federation/accreditation body for SPCAs/humane societies.
- This supports standards and sector coordination, but not a singular nationwide consumer listing index replacing local/aggregator pipelines.

## Individual Dog Temperament Data: What appears available now

Observed on Petfinder listing cards/search contexts:

- age category (puppy/young/adult)
- sex
- breed or breed mix
- shelter/rescue org name
- location context
- compatibility cues (`good with dogs/cats/children`)
- house-trained indicator

Likely available at greater depth on detail pages or partner APIs (to verify in implementation phase):

- descriptive bio text
- medical status fields (spayed/neutered/vaccinated)
- size/weight class
- behavior notes and handling requirements

## Data Quality & Matching Implications

Because individual temperament is shelter-observed and non-standardized:

- store raw source values and normalized values side-by-side
- keep confidence metadata (`observed`, `inferred`, `unknown`)
- timestamp behavior observations
- separate breed tendencies from individual behavior

## Suggested Implementation Pattern for This Reality

1. Ingest from 2-4 channels first (API where possible).
2. Canonicalize into one `DogListing` model with source provenance.
3. Deduplicate by combination of:
   - source IDs
   - fuzzy text + location + image hash heuristics
4. Retain channel-specific fields in a JSON extension column.
5. Track listing freshness and stale-out rules.

## Practical Conclusion

Treat US/Canada adoption data as a **federated network with partial overlaps** rather than a unified state registry. Your matching app can create strong value by being the normalization and ranking engine across that fragmented reality.
