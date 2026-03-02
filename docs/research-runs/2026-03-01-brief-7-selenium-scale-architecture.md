# Reference architecture diagram (text description)

```text
                          ┌─────────────────────────────────────────────────────────┐
                          │                 Control Plane                            │
                          │                                                         │
                          │  Source Registry + Policy Engine                         │
                          │  - source metadata (tier, schedule, rate limits)         │
                          │  - selector/extraction spec versions                     │
                          │  - promotion/demotion decisions                          │
                          │                                                         │
                          │           ┌─────────────────────────┐                   │
                          │           │ Workflow Orchestrator    │                   │
                          │           │ (daily schedules +        │                   │
                          │           │ backfills + retries)      │                   │
                          │           └───────────┬─────────────┘                   │
                          └───────────────────────┼─────────────────────────────────┘
                                                  │ emits “SourceRun” jobs
                                                  ▼
                               ┌───────────────────────────────┐
                               │    Job Queue / Router         │
                               │  - priority lanes (P0/P1/P2)   │
                               │  - per-source concurrency caps │
                               │  - DLQ / poison isolation      │
                               └───────────────┬───────────────┘
                                               │ claims jobs (at-least-once)
                                               ▼
┌───────────────────────────────────────────────────────────────────────────────────────────┐
│                                     Data Plane                                            │
│                                                                                           │
│  ┌─────────────────────────────┐          ┌───────────────────────────────────────────┐  │
│  │ Tiered Acquisition Workers  │          │       Browser Execution Fabric             │  │
│  │ (Kubernetes deployments)    │          │   (Selenium Grid + browser nodes)         │  │
│  │  - API/Feed fetchers        │          │  Router → New Session Queue → Distributor │  │
│  │  - Selenium scrapers        │◀────────▶│  → Session Map ↔ Nodes (Chrome/Firefox)   │  │
│  │  - Manual task hooks        │          │  + Grid status/observability endpoints    │  │
│  └─────────────┬───────────────┘          └───────────────────────────────────────────┘  │
│                │ produces artifacts + extracted records                                   │
│                ▼                                                                          │
│  ┌─────────────────────────────┐          ┌───────────────────────────────────────────┐  │
│  │ Artifact & Provenance Store │          │  Normalize + Validate + Dedupe            │  │
│  │ (object storage)            │          │  - schema validation & coverage checks    │  │
│  │  - HTML/WARC snapshots      │          │  - entity resolution / stable IDs         │  │
│  │  - screenshots              │          │  - publish “pet records” & deltas         │  │
│  │  - HAR/network logs         │          └───────────────────────┬───────────────────┘  │
│  │  - run manifests (PROV-like)|                                  │                      │
│  └─────────────┬───────────────┘                                  ▼                      │
│                │                                          ┌───────────────────────────┐  │
│                │                                          │ Clearinghouse Data Stores │  │
│                │                                          │ - operational DB          │  │
│                │                                          │ - warehouse/lake          │  │
│                │                                          └───────────────────────────┘  │
│                ▼                                                                          │
│  ┌─────────────────────────────┐                                                         │
│  │ Observability & Ops          │                                                         │
│  │ - metrics + tracing + logs   │                                                         │
│  │ - breakage detection         │                                                         │
│  │ - canary/synthetic monitors  │                                                         │
│  └─────────────────────────────┘                                                         │
└───────────────────────────────────────────────────────────────────────────────────────────┘
```

This design separates control-plane scheduling from data-plane execution so that “hundreds of source jobs/day” is primarily a queueing, policy, and concurrency-management problem—not a monolithic Selenium problem. The browser layer is treated as a scarce compute resource accessed via the WebDriver protocol (language- and platform-neutral remote control interface for browsers). citeturn0search1turn0search5

At the browser layer, Selenium Grid’s internal components (Router → New Session Queue → Distributor → Session Map → Nodes) already implement session routing and a FIFO “new session” queue with timeouts, which is useful for smoothing bursts of browser session requests. citeturn0search0turn0search4turn0search34

At the platform layer, Kubernetes Horizontal Pod Autoscaling provides a standard mechanism to scale worker deployments horizontally based on observed metrics (resource or custom/external), which is the simplest operational model for fluctuating crawl demand and varying source complexity. citeturn0search2turn0search6

For queueing semantics, assume at-least-once delivery and design for idempotency (dedupe at write time), because common managed queues explicitly use mechanisms like visibility timeouts where a message can reappear if not deleted before the timeout. citeturn2search3turn2search11turn2search15

# Component responsibilities

**Source Registry + Policy Engine (system of record for “what to ingest, how, and when”)**  
This service owns: (a) source identity, access method, and crawl targets; (b) “source policy tier” selection (API/feed/scrape/manual); (c) crawl budget envelopes; and (d) selector/extraction specs and their versions. Storing these as versioned configuration is a reliability requirement in the same way “configuration and binary changes introduce risk”; a config-only change (e.g., selector update) should be treated like a release and rolled out gradually. citeturn3search12turn3search0

**Crawler orchestration and queue strategy (hundreds of source jobs/day)**  
A workflow orchestrator should (1) compute the set of due “SourceRun” jobs (based on schedule, freshness needs, and backfills) and (2) enqueue them with explicit priority and concurrency constraints. Airflow’s scheduler model—monitor DAGs and trigger task instances when dependencies are satisfied—maps well to “run per source” workflows and supports retries/backfills as first-class concepts. citeturn2search4turn2search0

A production queueing strategy for this scale is typically two-layered:

- **Job queue (SourceRun-level):** one message per source run, with metadata (source_id, tier, priority, run_type=incremental/full, budget, config versions). Use at-least-once semantics and include a dead-letter queue (DLQ) to isolate “poison” runs that repeatedly fail. citeturn2search7turn2search11  
- **Optional page/task queue (Page-level):** only if a single SourceRun fans out into many independent pages. If you need that, partition by source_id so you can enforce per-source concurrency and avoid one source starving others.

If you implement the Job queue on a managed queue like SQS, explicitly tune: (a) **visibility timeout** to exceed worst-case run time (or implement heartbeat/visibility extensions), and (b) **DLQ redrive policy** for repeated failures. citeturn2search3turn2search7 If you implement on Kafka, accept that default delivery is at-least-once and plan for duplicates unless you add transactional/exactly-once patterns. citeturn2search2turn2search18

**Tiered Acquisition Workers (API / feed / scrape / manual)**  
Workers are stateless executors that claim jobs from the queue and emit (a) extracted records and (b) provenance artifacts. They should be horizontally scalable via Kubernetes HPA so “more due jobs” and “more expensive pages” translate into more pods rather than larger pods. citeturn0search2

Workers should be tier-specialized:

- **API worker:** fetches official endpoints, handles auth, and enforces provider rate limits (best-effort compliance).  
- **Feed worker:** consumes structured feeds (partners, exports, or publisher-provided listings) with incremental checkpoints.  
- **Scrape worker:** uses Selenium/WebDriver for dynamic sources.  
- **Manual worker hook:** creates a human task with all artifacts needed to reproduce and capture data when automation is blocked.

**Browser execution fabric (Selenium Grid + autoscaled browser nodes)**  
Treat browsers as a shared cluster resource. Selenium Grid provides distributed routing and session lifecycle primitives and exposes a status endpoint for fleet-level monitoring. citeturn5search2turn5search25

Key responsibilities:

- Provide standardized WebDriver sessions (capabilities/options) per source class (e.g., headless mode, viewport size, locale, user-agent policy). WebDriver sessions and capabilities are core to the W3C protocol and browser-driver interoperability. citeturn0search5turn0search1turn0search17  
- Absorb spikes in session requests via the Grid’s “New Session Queue.” citeturn0search0turn0search34  
- Emit trace data: Selenium server is instrumented with OpenTelemetry tracing “from start to end” for requests, enabling cross-component diagnosis when correlated with worker/job traces. citeturn5search5turn1search2  
- Optionally scale nodes based on pending session queue using autoscaling patterns (e.g., KEDA-triggered scaling in some Helm deployments), if you want Grid-driven scale signals rather than only worker-driven signals. citeturn5search18

**Selector/extraction specification management (the “contract” between each source UI and your data model)**  
Selector specs should be treated as controlled, testable artifacts:

- **Representation:** a per-source “ExtractionSpec” containing selectors (CSS/XPath), parsing rules, normalization transforms, and validation assertions. Selenium supports standard locator strategies (including CSS selector and XPath). citeturn1search4turn1search0  
- **Style guidance:** prefer stable, well-written CSS selectors when unique IDs are unavailable; XPath is powerful but can be harder to debug and can have performance pitfalls in practice. citeturn1search12  
- **Versioning:** every SourceRun stores references to the exact ExtractionSpec version used, so you can reproduce regressions and roll back “selector releases” quickly (a release engineering principle: repeatable, non–snowflake changes). citeturn3search24turn3search12

**Breakage detection and canary test strategy (make failures cheap and early)**  
Implement three layers of detection:

- **Pre-merge/unit tests (fast):** run ExtractionSpec tests against stored HTML fixtures and a small set of recorded sessions.  
- **Canary runs (real browsers, production-like):** “canary” new selectors/configs by applying them to a small segment of sources/traffic before broad rollout—mirroring canary release principles where you limit blast radius by testing on a small fraction first. citeturn3search0turn3search12  
- **Synthetic monitoring (continuous):** run scripted browser transactions on a fixed schedule to detect availability/flow breakage before it impacts the full ingestion schedule; synthetic monitoring is explicitly about scripted checks simulating user journeys. citeturn3search1turn3search9turn3search13

Breakage detection signals should include: selector exceptions, step-timeouts, sudden drops in extracted record counts, and schema coverage regressions (e.g., “breed missing in 70% of records”). Using explicit waits is the recommended technique to synchronize with dynamic page state; without it you’ll see timeouts or stale element behaviors more frequently. citeturn6search0turn6search11

**Source policy tiers and promotion/demotion rules (API/feed/scrape/manual)**  
A pragmatic tier model for a pet data clearinghouse is:

- **Tier A — API:** official or partner APIs (highest reliability/lowest compute).  
- **Tier B — Feed/export:** structured partner exports, syndication feeds, or bulk dumps (moderate reliability/low compute).  
- **Tier C — Scrape (Selenium):** dynamic websites with no viable structured interface (highest compute/highest breakage rate).  
- **Tier D — Manual:** human-assisted capture for blocked sources (CAPTCHA, legal restrictions, highly brittle pages).

Policy should be enforced by an explicit rules engine that consults both technical and compliance constraints. For example, crawlers are requested to honor robots exclusion rules (REP is standardized in RFC 9309) and this should be part of “allowed surfaces” evaluation. citeturn4search2

Promotion/demotion should be metric-driven:

- **Promote C → B/A** if repeated scraping breakage occurs and you discover stable underlying endpoints (often identifiable via network inspection artifacts like HAR logs), or if a partner offers a feed/API. HAR is a standard JSON archive format for HTTP transactions and is commonly used to capture request/response timelines. citeturn3search2turn3search23  
- **Demote A/B → C** only when API/feed coverage drops below a threshold or becomes stale (e.g., missing new records), and only if policy permits scraping.  
- **Demote C → D** when automation is consistently blocked (e.g., persistent 403/429, interstitial challenges) or when REP/terms indicate the relevant URL space should not be crawled.

Rate-limit signals should be interpreted explicitly: HTTP 429 is defined as “Too Many Requests” in the additional HTTP status codes RFC and is a standard indicator that clients should slow down; servers may provide Retry-After guidance. citeturn4search3turn4search7

**Audit, provenance, and artifact storage (reproducibility + compliance + debugging)**  
Every SourceRun should produce a “run manifest” plus raw evidence. Provenance is formally modeled as relationships between entities, activities, and agents (W3C PROV). citeturn0search3turn0search19

Suggested artifacts (store pointers in the manifest; store large blobs in object storage):

- **Run manifest:** source_id, run_id, timestamps, worker identity, browser/driver versions, IP/proxy identity, orchestrator job id, and config references (ExtractionSpec version, code version). (This is how you make sources reproducible and comparable across time.) citeturn0search19turn3search24  
- **Raw content snapshot:** store full HTML and/or WARC. WARC is an ISO standard specifically intended for aggregating harvested web resources with related information, widely used to store web crawls. citeturn4search1turn4search32  
- **Screenshots:** store at least one “final state” screenshot and optionally step-level screenshots when failures occur; Selenium provides a standard screenshot interface. citeturn6search2  
- **Network capture:** store HAR (requests/responses, timing) for debugging hidden XHR endpoints and diagnosing data gaps; HAR 1.2 is a documented format for exporting captured HTTP data. citeturn3search2turn3search18  
- **Logs and security audit trail:** centralize and retain logs with integrity controls and retention policies; log management guidance is a recurring requirement in security programs (NIST SP 800-92). citeturn1search3turn1search7

# Failure modes and mitigations

**Failure mode: queue duplicates and “stuck” jobs**  
At-least-once queues can deliver duplicates (e.g., if a message visibility timeout expires before deletion), which can produce double-ingestion unless you design idempotent writes. Mitigation: write extracted records using deterministic keys (source_id + source_native_id + observation_date) and enforce dedupe at the storage boundary; configure visibility timeouts to exceed worst-case processing and use DLQs for repeated failures. citeturn2search3turn2search7turn2search11

If using Kafka, offset management choices directly affect delivery semantics, with at-least-once being the default assumption; mitigation remains idempotency/deduplication unless you adopt transactional patterns. citeturn2search2turn2search18

**Failure mode: Selenium Grid saturation / session starvation**  
Grid has its own “New Session Queue” and will queue session requests; however, if nodes are at capacity, job latency increases and can cascade into orchestrator retries. Mitigation: instrument queue depth and session wait time via Grid status endpoints, and autoscale browser nodes and/or workers to maintain headroom. citeturn0search0turn5search2turn0search2

**Failure mode: dynamic UI timing issues (stale elements, timeouts, partial rendering)**  
Dynamic sites often mutate the DOM after initial page load, which can lead to stale-element failures or element-not-found errors. Mitigation: standardize on explicit waits for specific conditions (rather than sleeps) because explicit waits poll until conditions are satisfied or time out, improving robustness for asynchronous states. citeturn6search0turn6search11

**Failure mode: selector breakage due to front-end redesigns**  
CSS/XPath locators can break when DOM structure changes. Mitigation is multi-pronged: (a) follow locator design guidance (prefer well-written CSS selectors when IDs aren’t available), (b) maintain fallback selectors and semantic assertions, and (c) detect breakage via canaries + synthetic monitors so you learn quickly. citeturn1search12turn3search9turn3search0

**Failure mode: rate limiting, blocking, and crawl policy violations**  
You will see 429 responses when you exceed server-defined rate limits. Mitigation: implement per-source rate limits, exponential backoff, and Retry-After honoring when present. citeturn4search3turn4search7

Separately, REP/robots rules are a standardized mechanism by which service owners request crawler behavior; mitigation is to evaluate REP before scheduling/crawling and enforce “allowed paths” in policy. citeturn4search2

**Failure mode: insufficient forensic evidence to debug extraction regressions**  
Without replayable artifacts, you cannot distinguish “site changed” from “automation bug” from “transient outage.” Mitigation: store WARC/HTML snapshots, HAR logs, and final screenshots for every run (or at least for every failed run) and make them discoverable by run_id. WARC is purpose-built for aggregating harvested web resources plus related metadata; HAR is a structured log of browser HTTP interactions. citeturn4search1turn3search2

**Operational KPIs (what to measure to keep the system reliable)**  
Use a combination of “SRE golden signals” and domain-specific ingestion metrics. Google’s four golden signals are latency, traffic, errors, and saturation—useful to structure dashboards and alerts for ingestion services. citeturn1search1

For a Selenium ingestion platform, translate them as:

- **Latency:** queue wait time; SourceRun duration; Grid session acquisition time. (Grid status endpoints + job metrics.) citeturn5search2turn2search4  
- **Traffic:** runs/day, pages/day, records/day per tier (API/feed/scrape/manual). citeturn1search1  
- **Errors:** per-source failure rate, selector exception rate, HTTP 4xx/5xx distribution (watch 429 spikes). citeturn4search3turn4search7  
- **Saturation:** browser node utilization, pending session queue depth, worker CPU/memory (Kubernetes HPA signals). citeturn0search2turn0search0

To make incidents debuggable end-to-end, propagate a run_id/trace_id across components; OpenTelemetry’s context propagation enables correlation across traces/metrics/logs. citeturn1search2turn1search10

**Incident playbooks (high-frequency operational scenarios)**  
Incident response works best with a defined process and up-to-date playbooks; Google’s incident guidance emphasizes preparedness, actionable alerting, and structured coordination. citeturn5search3turn5search9

Minimal playbooks to implement for this architecture:

- **Selector breakage spike (many runs failing for one source family):** freeze config rollout (treat selectors as a release), run canary extraction against stored artifacts, patch ExtractionSpec, and re-canary before broad deployment. Canarying changes reduces risk by exposing changes to a small segment first. citeturn3search0turn3search12  
- **Grid saturation / backlog:** check /status for node availability and session capacity, scale browser nodes/workers, and temporarily lower crawl concurrency for low-priority sources. citeturn5search2turn0search2  
- **Rate limit / blocking event (429/403 surge):** immediately reduce rate, honor Retry-After, rotate to allowed access methods (API/feed), and if blocked persists promote manual capture for critical sources while policy is reviewed. citeturn4search7turn4search3turn2search11  
- **Data-quality regression (records drop or field coverage collapse):** trigger “quality gate” to prevent publishing bad deltas; inspect run artifacts (WARC/HTML + HAR + screenshot) to determine whether the root is UI change or logic bug. citeturn4search1turn3search2  
- **Security/audit event (missing logs or integrity concerns):** follow centralized log management practices and retention/integrity controls; NIST guidance explicitly treats log management as an enterprise security practice. citeturn1search3turn1search7

Close the loop with blameless postmortems for material incidents; blameless postmortems focus on contributing factors and system improvements rather than individual fault. citeturn5search1turn5search4

# MVP implementation plan

**Milestone: define the ingestion contract (week one)**  
Establish a minimal, explicit “SourceRun” contract: source_id, tier, schedule, rate limit, and an ExtractionSpec identifier. This is the smallest unit your orchestrator and queue understand, and it is what enables reproducibility and controlled rollouts (treat config like a release). citeturn3search24turn3search12

**Milestone: stand up the execution fabric (weeks one to two)**  
Deploy Selenium Grid (hub/distributed) and validate health visibility via the Grid status endpoint. Grid’s architecture and endpoints are documented and should be operationally verified early. citeturn0search4turn5search2  
Deploy Kubernetes worker deployments with HPA enabled (even if initial scaling bounds are conservative), since horizontal scaling is the primary mechanism for matching capacity to demand. citeturn0search2

**Milestone: implement the queue and idempotent write path (weeks two to three)**  
Choose and implement a job queue with DLQ support and visibility/timeout semantics. If using SQS-like semantics, build job processing around visibility timeouts and DLQ isolation. citeturn2search3turn2search7  
Implement idempotent publishing/deduplication because at-least-once delivery implies occasional duplicates. citeturn2search11turn2search15

**Milestone: implement selector specs + baseline extraction library (weeks three to four)**  
Create the ExtractionSpec schema and a reference implementation supporting Selenium locator strategies (CSS/XPath) and standardized waiting patterns. Selenium documents locator strategies and explicit waits as core mechanisms. citeturn1search0turn6search0  
Start with 5–10 representative dynamic sources (the “high-value” set) and build stable extraction with strong validations (expected fields, record count bounds).

**Milestone: add provenance artifacts from day one (weeks three to five)**  
Write run manifests and store a reproducible artifact bundle per run: HTML/WARC snapshot, screenshot, and (for debugging) HAR logs. WARC is an ISO standard for web crawl archiving; HAR is a structured HTTP archive format. citeturn4search1turn3search2  
Centralize logs and define retention/integrity controls consistent with enterprise log management guidance. citeturn1search3

**Milestone: canary + synthetic monitoring (weeks five to six)**  
Implement:

- **Canary pipeline** for selector/config changes: apply to a small subset first and automatically compare key metrics (success rate, coverage, record counts) against control. Canarying is explicitly designed to reduce rollout risk by limiting exposure initially. citeturn3search0turn3search12  
- **Synthetic monitors** for critical sources: scripted Selenium transactions on a schedule to detect login/navigation/listing path breakage proactively; synthetic monitoring is defined as scripted checks simulating user actions. citeturn3search9turn3search13

**Milestone: operational readiness (weeks six to eight)**  
Define SLIs/SLO-aligned alerts using the golden signals framework (latency/traffic/errors/saturation) adapted to ingestion. citeturn1search1  
Adopt an incident process and require playbooks for the top scenarios; Google’s incident management guidance emphasizes preparation, actionable alerting, and documented procedures. citeturn5search3turn5search9  
Require postmortems for Sev-1/Sev-2 incidents and track action items; blameless postmortem culture is presented as a resilience practice. citeturn5search1turn5search4