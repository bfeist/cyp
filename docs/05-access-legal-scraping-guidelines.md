# Access Strategy, Licensing, and Scraping Guidelines

_Last updated: 2026-03-01_

## Purpose

This document sets a practical ingestion policy for the prototype so we can move fast **without creating avoidable legal/compliance risk**.

## Operating Policy (Recommended)

## Rule 1: API/official feeds first

Prefer:

1. Official API
2. Written partnership feed/export
3. Public dataset with explicit license

Only consider scraping if:

- no practical API/feed exists
- terms allow it (or do not prohibit with legal confirmation)
- robots and rate limits are respected
- data scope is minimized

## Rule 2: Keep source provenance for every field

Every ingested field should carry:

- source URL
- timestamp
- license status (`allowed`, `restricted`, `unknown`)

## Rule 3: Separate “reference use” from “bulk extraction”

Even if content is publicly viewable, that does not imply extraction/republication rights.

---

## High-impact restrictions identified in this research pass

## AKC

Observed language indicates:

- prohibition on robots/scrapers/data extraction tools
- prohibition on automated screen scraping for commercial purposes
- reproduction restrictions for online material, with permission workflows required

Action:

- do not bulk scrape AKC pages for product data without explicit authorization.
- if needed, pursue written permission path described in AKC policies.

## OFA

Observed language indicates:

- OFA content/database search results are copyrighted
- public reproduction prohibited without express permission

Action:

- treat OFA as partnership/licensing-required for database-level integration.

## Shelter platform ecosystems

- RescueGroups: explicit API program and key request flow (strong candidate).
- Shelterluv: platform terms include restrictions around misuse and automated high-volume behavior; treat as partner integration.

## Petfinder / Adopt-a-Pet docs ambiguity

- Some historical developer URLs were inaccessible via automated fetch.
- Public listing pages are accessible.

Action:

- direct manual verification of current developer program and contract terms before systematic ingestion.

---

## Ingestion Risk Matrix (Prototype)

- **Low risk**: Official API with key + documented terms + stored attribution.
- **Medium risk**: Public web pages with ambiguous terms, low-volume metadata extraction only, pending legal confirmation.
- **High risk**: Explicit anti-scraping terms + bulk extraction + republication of protected content.

## “Can we scrape it?” checklist

Before writing any crawler:

1. Is there an API/feed/partner alternative?
2. Do terms explicitly prohibit scraping, bots, extraction, or commercial reuse?
3. Do we need to republish source text/images?
4. Can we reduce to links/IDs and user-side redirect instead?
5. Has legal reviewed high-volume collection plan?

If any answer signals high risk, stop and escalate.

## Prototype-safe implementation pattern

- Build ingestion connectors behind feature flags.
- Start with explicitly allowed/API-based sources.
- For blocked sources, store only:
  - source URL templates
  - manual-curation entries
  - outbound-link user journeys

## Compliance guardrails to implement in code

- Connector-level rate limiter
- robots.txt awareness (for web connectors)
- audit logs for crawl/fetch events
- source takedown switch (per source)
- retention policy and deletion workflow

## Partnership outreach priority list

1. RescueGroups API (high practical value)
2. Petfinder developer/partner program (if active/current)
3. Shelter software vendors (Shelterluv and peers)
4. Registry/health data permission channels (AKC/CKC/OFA where needed)

## Not legal advice

This document is a technical research guide. Final production decisions should include legal review in your target jurisdictions and business model context.
