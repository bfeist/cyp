# Dog Match Data Prototype Research Pack (US + Canada)

_Last updated: 2026-03-02 (deep research runs completed)_

This docs folder is a seed package for building a **dog-owner ↔ dog-breed matching application** with strong data foundations.

## Start here

> Six of seven deep research runs are complete. Read `00-executive-summary.md` first — it consolidates all confirmed decisions, what changed from prior assumptions, and the immediate build priorities.

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

- **`00-executive-summary.md` — start here.** Consolidated decisions from all research runs, what changed vs assumptions, and immediate build priorities.
- `01-data-landscape-us-canada.md` — executive summary of data landscape and key findings (updated with research findings).
- `02-source-catalog.md` — source-by-source inventory with confirmed access status (updated with research findings).
- `03-adoption-indexing-us-canada.md` — deep dive on national vs regional indexing and adoption-profile data.
- `04-data-model-proposal.md` — normalized schema proposal for prototype ingestion and matching.
- `05-access-legal-scraping-guidelines.md` — practical/legal access strategy and risk matrix.
- `06-research-backlog-and-ai-agent-briefs.md` — iterative backlog + copy/paste prompts for external deep research sessions; tracks completed runs and open questions.
- `07-clearinghouse-api-architecture.md` — system architecture for becoming a unified data/API clearinghouse (updated with confirmed connector order and Brief 7 findings).
- `08-human-profile-modeling-research.md` — third-pillar research plan for advanced human profiling (updated with confirmed trait ontology, event schema, and model roadmap).
- `research-runs/` — raw deep research session outputs.

## Important Framing

- This package is intentionally designed as a **research and prototyping foundation**.
- For production use, prioritize: API contracts, data licensing, provenance tracking, and legal review.
- Some vendors/documentation pages were partially inaccessible from automated fetches; those are flagged with follow-up tasks rather than guessed.
