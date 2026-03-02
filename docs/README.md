# Dog Match Data Prototype Research Pack (US + Canada)

_Last updated: 2026-03-01_

This docs folder is a seed package for building a **dog-owner ↔ dog-breed matching application** with strong data foundations.

## Goals

1. Identify high-value data sources for:
   - Breed-level traits (temperament, size, energy, health predispositions)
   - Individual-animal adoption data (behavior, compatibility, house-training, location)
   - Breeder availability data (purchase pathway)
   - Human-profile traits required for high-quality owner↔dog matching
2. Clarify whether a national adoption index exists in the US/Canada.
3. Map practical ingestion options: official APIs, exports, partnerships, and large-scale scraping paths.
4. Propose a v1 canonical data model and research backlog.

## Document Map

- `01-data-landscape-us-canada.md` — executive summary of data landscape and key findings.
- `02-source-catalog.md` — source-by-source inventory with recommended usage path.
- `03-adoption-indexing-us-canada.md` — deep dive on national vs regional indexing and adoption-profile data.
- `04-data-model-proposal.md` — normalized schema proposal for prototype ingestion and matching.
- `05-access-legal-scraping-guidelines.md` — practical/legal access strategy and risk matrix.
- `06-research-backlog-and-ai-agent-briefs.md` — iterative backlog + copy/paste prompts for external deep research sessions.
- `07-clearinghouse-api-architecture.md` — system architecture for becoming a unified data/API clearinghouse.
- `08-human-profile-modeling-research.md` — third-pillar research plan for advanced human profiling beyond questionnaires.

## Important Framing

- This package is intentionally designed as a **research and prototyping foundation**.
- For production use, prioritize: API contracts, data licensing, provenance tracking, and legal review.
- Some vendors/documentation pages were partially inaccessible from automated fetches; those are flagged with follow-up tasks rather than guessed.
