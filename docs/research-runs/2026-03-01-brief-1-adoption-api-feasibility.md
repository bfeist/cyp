# Brief 1 — Petfinder/Adoption API Feasibility

## Executive summary

As of **2026-03-01**, **Petfinder’s public developer API is not a viable production ingestion source**: Petfinder’s own support materials state the **Petfinder API was decommissioned after 2025-12-02**, and Petfinder directs users to a **“Custom Pet List” embeddable widget** as the supported replacement. citeturn31search1turn31search0turn31search4 In practical terms, the former developer documentation endpoints now return **404 Not Found**, reinforcing that **new integrations should not assume any supported JSON API access**. citeturn5view0turn5view1turn5view2

For production-quality programmatic access today, the strongest “open-ish” option among the three sources is **RescueGroups**: it provides a documented **v5 REST API** supporting public adoptable pet and organization search via an **API key** (Authorization header), with explicit guidance on headers, pagination, maximum per-request limits, and rate-limit behavior (returns **429** under abnormal load; no published quota). citeturn33view0turn33view2 However, RescueGroups’ **API Terms** impose important operational and product requirements—most notably: (1) **data rights limited to temporary use/display** (with “temporary caching” allowed), (2) **organization-level opt-in/out**, (3) **required “Pet Adoption Tracker” image** on every public pet detail page (unless private/educational), and (4) **removal SLAs** (ability to remove an organization within 1 business day; deletion on termination). citeturn33view2

**Adopt-a-Pet’s data access is primarily partnership-gated**. Adopt a Pet explicitly states it provides APIs to **“select partners”** and that it **cannot provide API keys to students or organizations outside partners**, instructing prospective partners to email them. citeturn6view1 The partner API PDFs describe a legacy-style **HTTP GET + querystring** interface keyed by an **API key (`key=`)** and version param (`v=`), with endpoints such as `pet_details`, `limited_pet_details`, `pets_at_shelter`, and `pets_at_shelters`, and a throttling-style failure mode (`503 resource_unavailable`). citeturn7view0turn30view0turn30view2 Separately, Adopt a Pet also offers a **free “Pet List API”** for *individual shelters/rescues* to embed their own pets (limited info; not public for general third parties). citeturn12search4

**Bottom line:** For an MVP that needs reliable, compliant programmatic ingestion without bespoke enterprise agreements, **RescueGroups is the only one of the three that is (a) publicly documented and (b) currently available for third-party programmatic search**. citeturn33view0turn33view2 For production expansion, you should design a **source-agnostic ingestion layer** and pursue **Adopt a Pet partner access** in parallel, while treating Petfinder as **widget-only / redirect-only** unless you negotiate a separate commercial arrangement. citeturn6view1turn31search1turn31search2

## Source-by-source comparison table

| Source | Current programmatic availability (2026-03-01) | Access model | Auth model | Rate limits / throttling signals | Representative endpoints / capabilities | Key usage constraints for a consumer app |
|---|---|---|---|---|---|---|
| entity["organization","Petfinder","pet adoption platform"] | **Public API: decommissioned** after 2025-12-02; replacement is a **Custom Pet List widget**. citeturn31search1turn31search0 Developer endpoints now 404. citeturn5view0turn5view1turn5view2 | Widget/embed for org sites; no supported public developer API for third-party data ingestion. citeturn31search2turn31search1 | Widget generated after Petfinder account login; public API auth not applicable now. citeturn31search2turn31search1 | Not applicable for a public API (decommissioned). citeturn31search1 | “Custom Pet List” embed (copy/paste HTML code; filter/sort options). citeturn31search2 | You can **embed** or **deep-link**, but cannot ingest data as a supported JSON API. Widget use is governed by Petfinder ToS. citeturn31search2turn31search1 |
| entity["organization","RescueGroups.org","animal welfare api provider"] | **Active, documented v5 API** (public and private). Public access requires an API key. citeturn33view0turn33view2 | Public API key for adoptable pets + orgs; bearer token flow for private data/actions. citeturn33view0 | `Authorization: <apikey>` for public endpoints; `Authorization: Bearer <token>` for token-secured endpoints; `Content-Type: application/vnd.api+json`. citeturn33view0 | No published quota; API returns **429 Too Many Requests** under abnormal request volume; caching encouraged. citeturn33view0 | REST search (views + filters), pagination, includes, max default limit rules (often **max 250**). Example shown: `/public/animals/search/available/dogs/?limit=10&page=2...`. citeturn33view0turn33view1 | You must comply with API Terms: “temporary use/display,” optional temporary caching, orgs can opt out, **Pet Adoption Tracker** required on public pet detail pages, removal obligations, delete on termination. citeturn33view2 |
| entity["organization","Adopt-a-Pet.com","pet adoption website"] | **Partner APIs available**, but **keys restricted to select partners**; not generally issued to non-partners/students. citeturn6view1 Also offers a shelter-only “Pet List API” embed with limited info. citeturn12search4 | Partner-only API (commercial relationship). Separate shelter/rescue login for “Pet List API” (per-shelter). citeturn6view1turn12search4 | Legacy-style querystring auth: `key=<api_key>` and version `v=...` on GET endpoints. citeturn7view0turn30view0turn30view2 | Throttle-style error code: **503 resource_unavailable** “too many requests.” No published numeric limits in the PDFs. citeturn30view2turn30view0 | Endpoint families in PDFs: `/search/pet_details`, `/search/limited_pet_details`, `/search/pets_at_shelter`, `/search/pets_at_shelters`. Also references `pet_search` and `search_form` capabilities. citeturn7view0turn30view0turn30view2turn9view0 | Biggest blocker is **access gating** (partner status). For non-partners, options are widgets/links or per-shelter embed keys—insufficient for a broad consumer search app. citeturn6view1turn12search4 |

## Endpoint and auth summary by source

**Petfinder**

Official docs/links (as they exist today):
```text
Petfinder API decommission notices (support):
https://help.petfinder.com/hc/s/article/Petfinder-Site-Upgrade-and-Maintenance-December-2025
https://help.petfinder.com/s/article/Custom-Pet-List-Widget-Transition-Guide
Custom Pet List widget landing page:
https://www.petfinder.com/tools-widgets/custom-pet-list/getting-started/
```
citeturn31search1turn31search0turn31search2

Current reality:
- **API availability:** Petfinder states the Petfinder API was **decommissioned after 2025-12-02**, and directs users to the **Custom Pet List Widget**. citeturn31search1turn31search0turn31search4  
- **Developer docs:** Historical `/developers` URLs now return **404 Not Found**. citeturn5view0turn5view1turn5view2  
- **Auth flow / rate limits / endpoints:** Not applicable as a supported public integration surface (because the public API is decommissioned). citeturn31search1turn31search0  
- **What you can still do (supported):** Use the **Custom Pet List** embeddable widget (“copy/paste code into your website HTML”) after signing into a Petfinder account; using it acknowledges Petfinder’s Terms of Service. citeturn31search2

Historical context (useful only for legacy audits/migrations):
- Petfinder’s own open-source JS SDK historically required an **API key + secret** and exposed a client that could do `.animal.search()`. citeturn16view0  
- This does **not** establish current availability; it only helps identify the legacy dependency footprint during migration planning. citeturn31search1turn5view1turn16view0

**RescueGroups**

Official docs/links:
```text
API docs (v5):
https://api.rescuegroups.org/v5/public/docs
API key request entry point:
https://rescuegroups.org/services/adoptable-pet-data-api/
API Terms of Service:
https://rescuegroups.org/api-terms-of-service/
```
citeturn33view1turn33view0turn33view2

Authentication + required headers:
- **Public data:** `Authorization: <API_KEY>` (API key auth) for public adoptable pet and organization search. citeturn33view0  
- **Private data:** `Authorization: Bearer <token>` for endpoints requiring a token; tokens are refreshed/rotated (an updated Authorization header may be returned and must be stored). citeturn33view0  
- **Content type:** `Content-Type: application/vnd.api+json` for requests. citeturn33view0  

Rate limiting:
- RescueGroups explicitly states it **does not advertise specific rate limits**, but will return **`429 Too Many Requests`** under abnormal use; it explicitly recommends caching frequently used static data (breeds, species, colors, etc.). citeturn33view0turn33view1  

Endpoints (examples taken from RescueGroups docs):
- **Search available dogs (GET with “view” path + query params):**
  - Example format shown:  
    `/public/animals/search/available/dogs/?limit=10&page=2&sort=+distance&fields[animals]=name&include=fosters` citeturn33view0  
- **Paging + limit behavior:** default first page is 25 if you omit page/limit; **max limit for most endpoints is 250** (with exceptions for small static datasets). citeturn33view0turn33view1  
- **Advanced search (POST to `search` endpoint):** supported via filters in request body (docs describe fieldName/operation/criteria). citeturn33view1  

Compliance-related requirements that affect endpoint consumption:
- Data rights are limited to “temporary use and display,” and caching is described as **temporary**. citeturn33view2  
- If publicly displaying pet detail pages, each page must include the **Pet Adoption Tracker image** (often already embedded in HTML descriptions). citeturn33view2  

**Adopt-a-Pet**

Official docs/links:
```text
Partner API overview (Help Center):
https://adoptapetcom.zendesk.com/hc/en-us/articles/41654139166107-Adopt-a-Pet-Partner-APIs
Pet List API (shelter/rescue-oriented):
https://adoptapetcom.zendesk.com/hc/en-us/articles/201629944-Pet-list-API-code-for-your-website
Partner APIs host (documentation portal):
https://partner-apis.adoptapet.com/
```
citeturn6view1turn12search4turn12search0

Access model + gating:
- Adopt a Pet states it provides APIs to **select partners**, and that it **cannot provide API keys** to students or organizations outside partners; to become a partner, you must email them. citeturn6view1  
- The “Pet List API” is positioned as a **per-shelter/rescue** embed with limited data and links back to Adopt a Pet, and it is **not available to the general public**. citeturn12search4  

Authentication + request style (from the partner API PDFs):
- **Global inputs** include:
  - `key=` (API key identifying the partner account). citeturn9view0turn30view0  
  - `v=` (API version; options include `"beta"`, `"1"` default, `"2"`). citeturn30view0  
- Request style is **HTTP GET** with a script name + querystring (not OAuth). citeturn9view0turn7view0  

Rate limiting / throttling:
- Error tables include `503 resource_unavailable` indicating the system received **too many requests** and will not complete the request. citeturn30view2turn30view4  

Key endpoints (from the PDFs) + examples:
- **Pet detail endpoints:**
  - `/search/pet_details?key=...&v=1&pet_id=...` citeturn30view3turn9view1  
  - `/search/limited_pet_details?key=...&v=1&pet_id=...` citeturn7view1  
- **Pets at shelter endpoints:**
  - `/search/pets_at_shelter?key=...&shelter_id=...&start_number=...&end_number=...&output=json` citeturn30view0turn9view3  
  - `/search/pets_at_shelters?...` (plural, multi-shelter). citeturn30view0  
- **Pagination constraints (notably large caps):**
  - `end_number` defaults and maximums are documented; the Pets-at-shelter PDF shows `end_number` max **25000**. citeturn30view0turn9view3  
- The PDFs also describe broader “search_form” and “pet_search” capabilities and note that “pet_search” maximum results increased (to 10,000) with pagination constraints (retrieve 500 at a time). citeturn9view0turn30view0turn9view2  

## Blockers and fallback options

Petfinder blockers:
- **Primary blocker:** The **public Petfinder API is decommissioned** (post-2025-12-02); there is no supported path to ingest Petfinder adoption data via a production API today. citeturn31search1turn31search0turn31search4  
- **Operational blocker:** Legacy developer endpoints are now **404**, so automated onboarding/key issuance flows and official endpoint docs cannot be relied upon. citeturn5view0turn5view1turn5view2  

Petfinder fallbacks:
- Use Petfinder’s **Custom Pet List widget** for web surfaces where embedding is acceptable (e.g., “adoptable pets” page), and treat Petfinder as a **traffic destination** rather than a data source. citeturn31search2turn31search1  
- If your product requires native/mobile experiences, use **deep links** to Petfinder pet detail pages rather than trying to replicate Petfinder inventory in-app (this avoids unsupported ingestion). citeturn31search1turn31search2  
- Treat any “unofficial endpoints” usage as a high-risk last resort: community posts indicate the widget may call internal endpoints (e.g., GraphQL bases), but these are not documented as public APIs and create ToS/compliance and breakage risk. citeturn31search17  

RescueGroups blockers:
- **Compliance/product requirement:** If you publicly display pet detail pages, you must include the **Pet Adoption Tracker image** per pet detail page (unless private/educational). This impacts web and mobile rendering (you must preserve the HTML description or implement equivalent tracker inclusion). citeturn33view2  
- **Data rights constraint:** RescueGroups’ terms state no rights beyond “temporary use and display,” with only **temporary caching** permitted—this affects how you architect long-lived warehousing, model training datasets, analytics retention, and third-party sharing. citeturn33view2  
- **Coverage variability:** Organizations can decide whether to share data with your service based on the info in your API key profile; inaccurate/misleading key metadata can lead to opt-outs and termination risk. citeturn33view2  
- **Platform risks:** No published numeric rate limit—your production scaling plan must tolerate **429s** and implement caching/backoff. citeturn33view0turn33view1  

RescueGroups fallbacks:
- Architect your MVP so **RescueGroups is one connector**, not the whole system: keep a “source adapter” pattern and a canonical schema, so you can add other feeds later without rewriting product logic. This is directly aligned with RescueGroups’ “separate key per service” expectation and removal requirements. citeturn33view0turn33view2  
- Build a “no-results” and “partial coverage” UX that gracefully handles regional gaps and org opt-outs (a product fallback, not a technical hack). citeturn33view2  

Adopt-a-Pet blockers:
- **Primary blocker:** APIs are **partner-only**; Adopt a Pet explicitly says it cannot provide keys to non-partners/students, which makes it infeasible to rely on for a general MVP without a business development path. citeturn6view1  
- **Non-partner limitations:** The shelter “Pet List API” is not general-purpose (one shelter account’s pets, limited fields, links back to Adopt a Pet). citeturn12search4  
- **Throttling transparency:** Docs expose a “too many requests” error code but do not publish quotas—production scaling requires conservative caching and retry/backoff. citeturn30view2turn30view4  

Adopt-a-Pet fallbacks:
- Pursue a formal **partner relationship** and design your pipeline so Adopt a Pet can be added later as a connector (similar to the RescueGroups adapter pattern). citeturn6view1turn7view0  
- If your initial supply is a small set of partner shelters/rescues, you can ask those organizations to use their **own** shelter tools/embeds while you validate consumer demand; but this does not scale into a true cross-shelter search product without partner-grade access. citeturn12search4turn6view0  

## Final recommendation

For a US/Canada pet-adoption matching app that needs production-quality, programmatic adoption listings **today**, do not plan on Petfinder API ingestion: Petfinder indicates the API was decommissioned after 2025-12-02, and current developer endpoints are unavailable (404). citeturn31search1turn31search0turn5view1 Treat Petfinder as **widget-only / outbound-only**, using the Custom Pet List widget where it fits your web surface area and deep-linking for everything else. citeturn31search2turn31search1

For the **MVP ingestion strategy**, start with **RescueGroups v5** as your primary programmatic source because it is publicly documented, supports search at runtime, and has a clear auth/header model (API key + JSON:API content type), while being explicit about rate-limit behavior (429) and pagination/max limits (often 250). citeturn33view0turn33view1 Build an MVP pipeline that (a) performs live queries with **aggressive caching** of static/reference data, (b) stores only what you can justify as “temporary caching” under their terms, and (c) implements deletion/removal controls and the Pet Adoption Tracker requirement in your UI for public pet detail pages. citeturn33view2turn33view0

For **production**, keep a source-agnostic architecture (connector per provider, normalized canonical schema, dedupe strategy, monitoring, and a compliance layer) and pursue **Adopt a Pet partner access** in parallel, since their partner APIs exist but are explicitly gated to select partners. citeturn6view1turn7view0 This two-track plan (RescueGroups-first, Adopt-a-Pet partnership second, Petfinder outbound/widget) is the safest way to ship an MVP while preserving a credible path to broader inventory coverage without relying on unsupported or brittle scraping of decommissioned interfaces. citeturn33view0turn31search1turn6view1