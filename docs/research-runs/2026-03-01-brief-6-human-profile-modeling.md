# Human Profile Modeling Framework for Dog-Owner Matchmaking

## Trait ontology and signal map

This framework assumes the app’s “match” problem is **prospective owner ↔ individual dog** (e.g., shelter, rescue, foster, or rehoming listings) with success defined as (a) *retention* (low return/relinquishment risk), (b) *human satisfaction / bond formation*, and (c) *dog welfare* (needs met, manageable stress, safe fit). Evidence from shelter-return studies consistently shows **behavior-related issues** and **household compatibility** (especially with existing pets) are among the most frequent drivers of unsuccessful outcomes, making them central to the profile dimensions and to how the system should explain matches. citeturn3search10turn3search2turn0search0turn1search4

### Outcome-relevant profile dimensions

A useful ontology is **three-layered**: (1) *hard constraints* (must be true), (2) *capacity & lifestyle fit* (can the household meet needs), (3) *preference & bonding style* (what tends to create satisfaction once constraints/needs are met). This mirrors what return-reason evidence suggests: “mismatch” is often not one variable, but a compound of expectations, behavior manageability, and household context. citeturn0search0turn3search10turn3search2

**Owner / household dimensions (human side)**  
- **Time & routine capacity**: hours alone per day, schedule regularity, travel frequency, willingness to do structured training, ability to provide exercise/enrichment (a primary driver of whether “normal dog behavior” becomes “problem behavior”). citeturn3search6turn1search4turn3search2  
- **Experience & handling skill**: novice vs experienced; prior work with fearful/under-socialized dogs; willingness to attend post-adoption training classes (post-adoption training is studied as a factor associated with retention/returns). citeturn0search12turn1search8  
- **Household composition**: kids, seniors, other dogs/cats, frequent visitors; tolerance for barking/mouthing/jumping; safety constraints. In adoption-return datasets, incompatibility with existing pets is repeatedly high for dogs, so “other pets” must be treated as a first-class dimension rather than a footnote. citeturn3search10turn1search0  
- **Constraints & health**: allergies/respiratory sensitivity, mobility limits, budget for medical/grooming, housing type and size. Allergies/owner health appear explicitly as return reasons (more often in cats, but still relevant as a constraint model category). citeturn3search10  
- **Expectation setting & support needs**: “what you expect the dog to be like in 2 weeks / 3 months” and “what problems you can tolerate while settling.” Return-within-month evidence highlights that early-stage expectation mismatch is a real failure mode, so the profile should represent expectations as an editable, monitored dimension (not a one-time question). citeturn0search0turn1search4  

**Dog dimensions (dog side)**  
Because in-shelter behavior measures can be noisy due to stress and context differences, dog traits should be stored with **confidence/uncertainty** and multi-source provenance, not as absolute labels. Multiple reviews and studies caution that shelter assessments have limitations for predicting post-adoption behavior (especially aggression), and policy guidance explicitly notes inconclusive predictive usefulness for aggression prediction. citeturn1search1turn1search5turn1search17  

Recommended dog-side trait groups (with examples of validated questionnaire constructs as inspiration rather than as a hard requirement):
- **Temperament/behavior factors**: fearfulness, dog-directed fear, separation-related behavior, excitability/energy, trainability/responsiveness to training, stranger-directed fear/aggression, owner-directed aggression risk indicators, attachment/attention-seeking. (These map well to common structured instruments such as C-BARQ and DPQ factor groupings.) citeturn3search4turn3search5turn3search17turn1search17  
- **Activity & enrichment needs**: energy, play style, stamina, recovery time; mismatch here is a common pathway to nuisance behaviors and early returns. citeturn1search4turn3search6  
- **Sociability & compatibility**: with dogs, cats, children, visitors; this should be explicitly represented because incompatibility with existing pets is a major return category. citeturn3search10turn1search0  
- **Care complexity**: grooming/coat care, common medical needs, medication routines, special diets; this is both a welfare driver and a “cost of ownership” driver, which relates to bond/relationship satisfaction constructs. citeturn3search19turn3search7  

**Relationship dimensions (dyadic)**  
Treat the match as its own object: “Owner A + Dog B” has emergent properties not reducible to either profile alone.
- **Bonding style fit**: desired “active engagement” vs “affectionate engagement,” tolerance of perceived ownership costs; relationship-quality measurement work decomposes the human–dog bond into multiple components, which can be used to define explainability categories (“why this match may feel satisfying”). citeturn3search7turn3search19  
- **Risk alignment**: a household’s capacity/skill vs the dog’s uncertainty/behavioral risk; the product should surface this explicitly to reduce expectation-related returns. citeturn0search0turn1search17  

### Signal map: explicit vs implicit, and what each is good for

Successful modeling typically blends two signal classes:

**Explicit questionnaire signals** (high precision per question, but sparse and biased)  
Strengths:
- Can capture **hard constraints** (allergies, housing, other pets) and **non-observable intent** (willingness to do training, tolerance for barking) that may never appear reliably in early implicit behavior. citeturn3search10turn0search12  
Weaknesses:
- Self-report is vulnerable to **social desirability bias**, poor calibration (novices underestimate time/training cost), and “hypothetical preference drift” (what people say before meeting dogs differs from what they choose). Early return research highlighting expectation mismatch motivates designing explicit questions as revisitable, not final. citeturn0search0turn1search4  

**Implicit behavioral signals** (high volume, but noisy and biased by UI exposure)  
Examples: listing views, dwell time, saves, shares, messaging shelters/fosters, scheduling meet-and-greets, application starts/completions, post-adoption check-in responses.  
Strengths:
- Captures **revealed preferences** and can adapt quickly (especially when combined with bandit exploration). citeturn2search4turn2search8  
Weaknesses:
- Click/swipe data is systematically biased by **position bias** and exposure effects; learning directly from it can amplify popularity loops unless corrected with unbiased learning-to-rank / propensity methods. citeturn4search6turn2search21turn2search9  

### Ontology-to-signal mapping table

| Trait group | Examples (what you model) | Best explicit signals | Best implicit signals | Ground-truth/label proxies |
|---|---|---|---|---|
| Hard constraints | allergies, other pets, housing, mobility | direct questions; required fields | abandonment of flows when constraints clash (but ambiguous) | application rejection reasons; return reasons mentioning constraints citeturn3search10 |
| Lifestyle capacity | time alone, exercise capacity, travel | “weekday routine,” “max hours alone,” “training willingness” | meet-and-greet scheduling latency; engagement with training/support content | retention at 30/90 days; training class attendance association citeturn0search12turn1search8 |
| Dog behavior needs & uncertainty | fearfulness, separation behaviors, excitability, trainability | structured dog intake; foster notes | adopter questions/messages; “save” vs “apply” divergence for anxious profiles | post-adoption surveys at 1 month; behavior-related return rates citeturn1search5turn3search10turn1search4 |
| Dyadic compatibility | child/pet fit; risk alignment | “household members” + “comfort with behavior challenges” | repeated engagement with similar dog archetypes; refusal patterns | incompatibility return reasons; early return timing patterns citeturn3search10turn1search4 |
| Bonding style | affectionate vs active engagement; perceived costs | short relationship goals prompts | post-adoption check-ins; content interactions (walking, enrichment) | validated bond constructs as inspiration for measurement citeturn3search7turn3search19 |

## Data model and event schema recommendations

A dog-owner matchmaking app needs a data model that supports: (a) **multi-source traits with uncertainty**, (b) **event-level learning** (for rankers and active learning), and (c) **user control and privacy compliance** (minimize collection, clear purposes, straightforward access/correction). Privacy principles emphasizing limiting collection/use/retention and maintaining accuracy strongly suggest building profile inference as an auditable, correctable layer rather than an opaque embedding-only system. citeturn4search0turn2search3turn2search11

### Core entities

- **User**: account-level identity and preferences.  
- **Household**: one user may represent a multi-person household; store household constraints and capacities here.  
- **Dog**: canonical dog record (may be shared across partners/shelters), with stable identifiers.  
- **Listing**: context-specific representation (location, availability, adoption requirements, fee).  
- **Traits** (generic typed object): `trait_type`, `value`, `confidence`, `source`, `timestamp`, `expires_at` (important for behavior traits that change with time in foster care or post-adoption). Evidence that shelter environments are stressful and that assessments have limited predictive power motivates storing behavior traits probabilistically with provenance. citeturn1search1turn1search17  
- **Interaction events**: immutable, append-only log powering analytics and learning-to-rank.  
- **Outcome events**: meet-and-greet attended, adoption completed, retention check-ins, return with reason categories. Return datasets show reasons and timing are essential to understanding failure modes (e.g., very early nuisance-behavior returns), so the schema should capture both. citeturn1search4turn3search10turn3search2  
- **Explanation artifact**: stored “why” payload presented to users, enabling later dispute/correction and offline audits; explainability evaluation surveys emphasize that explanation quality itself must be testable, not just generated. citeturn1search14turn1search18  

### Event schema (recommended MVP)

Below is a practical event set that supports: baseline rules, ML rankers, counterfactual/bias correction, conversational elicitation, and profile correction UX.

| Event name | Actor | Required fields | Optional fields | Primary purpose |
|---|---|---|---|---|
| `profile_set_constraint` | user | `constraint_type`, `value`, `source="explicit"` | `confidence=1.0` | enforce hard filters; consented personal data handling citeturn4search0turn2search3 |
| `profile_set_preference` | user | `preference_type`, `value` | `strength`, `context` | initialize ranker priors; cold start |
| `trait_inferred_update` | system | `trait_type`, `value`, `confidence`, `evidence_event_ids` | `model_version` | implicit profile building; must be correctable for accuracy principles citeturn4search0turn2search3 |
| `dog_view` | user | `dog_id`, `listing_id`, `position`, `surface` | `dwell_ms` | implicit feedback with exposure logging for bias correction citeturn4search6turn2search21 |
| `dog_save` | user | `dog_id`, `listing_id` |  | strong positive signal |
| `dog_dismiss` | user | `dog_id`, `listing_id` | `dismiss_reason` | negative preference + explanation improvement loop |
| `message_thread_start` | user | `counterparty_type`, `listing_id` |  | high-intent signal |
| `meet_and_greet_scheduled` | user | `dog_id`, `datetime` | `location_type` | mid-funnel label |
| `application_started` | user | `dog_id`, `listing_id` |  | label for conversion modeling |
| `application_submitted` | user | `dog_id`, `listing_id` | `self_reported_commitment_score` | strong label; interpret with policy constraints |
| `adoption_completed` | partner/admin | `dog_id`, `user_id`, `date` | `channel`, `fee_bucket` | primary success label |
| `post_adoption_checkin` | user | `days_since_adoption`, `rating` | `issues[]`, `time_cost_delta` | welfare + satisfaction; captures expectation mismatch patterns citeturn0search0turn1search4 |
| `return_reported` | partner/admin | `dog_id`, `days_since_adoption`, `reason_code` | `free_text` | failure label with taxonomy aligned to literature (behavior, pet incompatibility, health/allergies) citeturn3search10turn3search2 |
| `preference_pairwise_choice` | user | `option_a_id`, `option_b_id`, `choice` | `question_strategy` | active learning / pairwise elicitation citeturn0search6turn4search7 |
| `explanation_shown` | system | `dog_id`, `explanation_id` | `user_action_after` | explanation A/B testing; “explainability affects behavior” measurement citeturn1search14turn1search2 |
| `profile_correction` | user | `trait_type`, `old_value`, `new_value` | `why_incorrect` | accuracy, trust, and compliance with correction/access expectations citeturn2search3turn4search0 |

### Data minimization and retention defaults

To align with widely used privacy principles (lawfulness/fairness/transparency; purpose limitation; data minimization; accuracy; storage limitation) and Canadian fair information principles, treat **each collection field** as needing a declared purpose and retention window, and separate “nice to have” personalization from “required for safety/fit.” citeturn4search0turn2search3turn2search7  

Recommended defaults:
- Store **raw message text** only if necessary for safety/support; otherwise store derived, user-visible tags with on-device or short-lived processing where possible (minimization). citeturn4search0turn2search3  
- Retain **event logs** in identifiable form briefly (e.g., 30–90 days) then aggregate/anonymize; maintain longer retention for audited aggregates and model evaluation. Storage limitation and limiting retention are explicit principles in major privacy frameworks. citeturn4search0turn2search7turn2search3  
- Log **exposure context** (`position`, `surface`) because unbiased learning-to-rank depends on knowing how items were presented; otherwise implicit feedback can be misleading. citeturn4search6turn2search21  

## MVP vs phase-2 model roadmap

A practical strategy is layered: **baseline rules for safety/constraints**, then an **ML ranker for personalization**, then an **online feedback loop** with exploration and bias correction. This is consistent with recommender-system practice where implicit feedback is abundant but biased, and where conversational/active elicitation methods help with cold start and sparse preference articulation. citeturn0search15turn0search7turn4search6turn2search4

### MVP: baseline rules + transparent scoring

**Filtering (hard constraints)**  
Implement deterministic constraint gates first (allergies, other pets compatibility requirements, housing constraints, mobility, maximum hours alone). These constraints directly address common mismatch pathways (pet incompatibility; health constraints) seen in return data. citeturn3search10turn3search2  

**Scoring (interpretable weighted rubric)**  
For remaining candidates, compute a “fit score” with a small set of human-readable components:
- **Needs coverage** (exercise/enrichment capacity vs energy/excitability)  
- **Risk alignment** (experience/training willingness vs fearfulness/separation behaviors uncertainty)  
- **Household compatibility** (kids/pets/visitors vs dog sociability indicators)  
- **Care complexity fit** (grooming/medical routines vs budget/time)  

Because behavior assessments can be uncertain and shelter stress can distort measurements, represent behavior-related features with confidence bounds and penalize overconfident gating based on a single assessment source. citeturn1search17turn1search1turn1search5  

### Phase-2: ML ranker for personalization with bias-aware learning

**Model family**  
Start with a learning-to-rank approach that can mix sparse explicit features and dense implicit signals (e.g., gradient-boosted decision trees over engineered features, or a two-tower retrieval model + shallow re-ranker). The important point is less the architecture than the **training signal design** and **bias correction** in implicit feedback. citeturn2search9turn4search6turn2search21  

**Training labels (multi-stage funnel)**  
Use graded labels such as:
- view (weak positive, exposure-biased)  
- save / message / meet-and-greet scheduled (stronger intent)  
- application submitted (high intent)  
- adoption completed (primary success)  
- retention at 30/90 days and return reason (long-term success/failure)  

Return studies show that reasons and timing matter (e.g., many nuisance behavior returns happen very quickly), so optimize not only “adoption conversion” but also downstream retention risk. citeturn1search4turn3search10turn3search2  

**Bias-aware implicit feedback learning**  
Clicks and engagement are biased by rank position and exposure. Unbiased learning-to-rank literature emphasizes propensity-based correction (e.g., inverse propensity scoring) and careful logging of exposure. citeturn4search6turn2search21turn4search2  

Practical MVP-to-phase-2 upgrade path:
- Add `position`/`surface` logging immediately (even in MVP) so later modeling has the needed context. citeturn4search6turn2search21  
- Use counterfactual estimators (IPS / doubly robust variants) when learning from clicks to reduce position bias and variance. citeturn4search2turn4search6  

### Adaptive preference elicitation: pairwise choices, conversation, active learning

**Pairwise choices (fast, low cognitive load)**  
Instead of asking “rate your preference for energetic dogs 1–5,” ask repeated micro-choices: “Which would you rather meet: Dog A or Dog B?” Pairwise preference learning methods model these comparisons (e.g., Bradley–Terry–style models) and active strategies can reduce the number of questions needed by choosing maximally informative comparisons. citeturn0search6turn4search7  

Implementation suggestion:
- Use an **active pairwise policy**: early on, ask comparisons that split uncertainty on the biggest drivers of mismatch (energy level; tolerance for fearfulness; pet compatibility). Active preference learning work demonstrates adaptive comparison selection can be markedly more sample-efficient than random comparisons. citeturn0search6turn0search2  

**Conversational prompts (structured extraction, with user control)**  
Conversational recommender surveys emphasize that dialog can be used both to **elicit constraints** and to **refine preferences** iteratively (“I want a calm dog… but also one that hikes”). citeturn0search15turn0search7  

A practical design:
- Use short prompts that map to ontology nodes, such as “Describe your weekday routine” → extracts `hours_alone`, `exercise_capacity`, `visitor_frequency`.  
- Immediately show extracted traits as editable chips (“You said: ‘alone ~6 hours/day’—edit?”) to reduce mis-inference and align with accuracy principles. citeturn4search0turn2search3  

**Active learning for cold start + disambiguation**  
Trigger questions when the system detects high posterior uncertainty that materially changes recommendations (e.g., user keeps saving high-energy dogs but claims “low energy”). Active learning then focuses on the highest expected-information-gain attribute, rather than running a long static questionnaire. This approach is consistent with active preference learning and conversational recommender framing. citeturn0search6turn0search7turn0search15  

### Online feedback loop: exploration, evaluation, and continuous improvement

**Exploration/exploitation**  
After an initial warm-start, use contextual bandit exploration (e.g., Thompson sampling variants) to occasionally surface diverse-but-plausible dogs, improving learning and avoiding over-commitment to early noisy signals. Exploration/exploitation trade-offs are the central motivation for contextual bandits in recommendation-like settings. citeturn2search4turn2search8  

**Counterfactual online learning-to-rank**  
To improve ranking continuously without destabilizing the user experience, combine online learning ideas with counterfactual evaluation so you can evaluate multiple candidate rankers and reduce reliance on expensive online A/B tests. Counterfactual online learning-to-rank work explicitly targets this combination. citeturn2search1  

**Metrics**  
Track a balanced scorecard:
- short-term: application submit rate, meet-and-greet completion  
- medium-term: adoption completion rate  
- long-term: retention at 30/90/180 days; return rate and reason distribution (especially behavior and pet incompatibility)  
- trust & UX: explanation helpfulness, correction rate, and “dismiss reason” coverage  

Return timing patterns (e.g., very early nuisance-behavior returns) imply that early follow-up and expectation-setting interventions should be measured as part of the loop, not treated as external to the recommender. citeturn1search4turn0search0  

### Explainability approach and profile correction UX

**Explainability goals**  
Explanations should be (a) **faithful** to the factors actually used, (b) **actionable** (what to prepare for), and (c) **editable** (if wrong). Explainable recommender research focuses on how explanations affect user understanding and behavior and underscores the need to evaluate explanation quality, not merely generate text. citeturn1search14turn1search2turn1search18  

**Explanation format (“Why this dog?”)**  
Use a consistent, ontology-aligned template:
- **Constraint match**: “Fits your allergy constraint and housing size.”  
- **Lifestyle fit**: “Energy level matches your 45–60 min/day walk plan.”  
- **Compatibility**: “Marked as comfortable with other dogs; you have a resident dog.”  
- **What to expect** (uncertainty-aware): “Some signs of separation stress reported in foster notes; recommended: gradual alone-time plan.”  

This aligns the app’s messaging with evidence that expectations matter for returns. citeturn0search0turn1search17  

**Correction UX (“Your profile, not our guess”)**  
- “Edit chips” everywhere: any inferred trait is shown with confidence (“We think you prefer high-energy dogs (70%).”) plus one-tap correction.  
- “Not relevant” and “Stop using this” controls at the trait level (prevents creepy-feeling over-personalization).  
- “I dismissed because…” capture lightweight labels to improve explanations and de-bias negative data.  
Accuracy and individual control principles are explicit in major privacy frameworks and help reduce user harm from misprofiling. citeturn4search0turn2search3turn2search11  

## Risks (bias/privacy) and mitigations

This category is not an afterthought: recommender systems can systematically amplify biases (via exposure loops and skewed data), and profiling systems raise privacy and compliance issues—especially when the system’s outputs could significantly affect users or partners. Fairness-aware recommender surveys catalog these issues and mitigation layers across the lifecycle. citeturn1search3turn1search11

### Bias and fairness risks

**Popularity and exposure bias (rich-get-richer dogs)**  
If a small subset of photogenic or already-popular dogs get most exposure, the system may reduce visibility for other adoptable dogs. Click-based learning-to-rank is known to be distorted by position bias without correction. citeturn4search6turn2search21turn1search11  

Mitigations:
- propensity/bias correction, plus explicit “exposure quotas” or diversity constraints in ranking; fairness-aware recommender work describes pre-, in-, and post-processing mitigation strategies. citeturn1search3turn1search11turn4search6  

**Socioeconomic proxy bias**  
Signals like device type, neighborhood, or work schedule can become proxies for protected attributes or class, potentially skewing match quality or access. Fairness-aware recommender research highlights risks of unfair exposure and allocation even when protected attributes are not explicitly used. citeturn1search3turn1search11  

Mitigations:
- feature governance (ban or bucket sensitive proxies), fairness testing on outcomes/exposure, and documentation of modeling choices; also consider bias-mitigation method suitability and legal constraints. citeturn1search3turn1search19  

**Dog labeling bias and measurement error**  
Shelter behavior assessments can be uncertain; overconfident labels (especially around aggression) can cause harmful wrong matches or unnecessary exclusion. Policy statements and empirical studies caution against assuming these assessments are highly accurate predictors, especially for aggression. citeturn1search17turn1search1turn1search5  

Mitigations:
- multi-source trait aggregation (foster notes + post-adoption check-ins), explicit uncertainty, conservative safety policies, and human review for high-risk cases. citeturn1search17turn1search5  

### Privacy, profiling, and compliance risks

**Overcollection and unclear purposes**  
Collecting extensive lifestyle/health data “for personalization” can violate data minimization and purpose limitation principles and increase breach harm. Data protection principles emphasize minimizing collection and limiting use/retention to stated purposes. citeturn4search0turn2search3turn2search7  

Mitigations:
- progressive disclosure (ask only when needed), explicit per-field purpose strings, short retention for identified logs, privacy reviews for new signals. citeturn4search0turn2search7turn2search3  

**Profiling concerns and user rights**  
Regimes like GDPR have specific provisions about automated decision-making and profiling with significant effects, emphasizing safeguards and human involvement in certain scenarios and requiring transparency. citeturn2search2turn2search6  

Mitigations:
- keep the product framed as a **decision support tool** (recommendations, not automated denial), provide meaningful explanations and easy appeal/correction paths, and document lawful basis/consent where required. citeturn2search2turn2search6turn4search0  

**Canadian privacy obligations (Toronto-relevant deployment)**  
Canadian private-sector principles emphasize consent, limiting collection/use/retention, safeguards, openness, and individual access—directly motivating the “profile correction UX” and data retention controls. citeturn2search3turn2search11turn2search7  

Mitigations:
- implement self-serve data access/export/delete, clear consent flows for sensitive data, strict access controls, and monitoring of internal access. citeturn2search7turn2search3  

**US privacy patchwork risk (if expanding beyond Canada/EU)**  
US state privacy laws (e.g., California) provide consumer rights such as opting out of certain sharing practices; expansion planning should assume jurisdiction-specific requirements. citeturn4search1  

Mitigations:
- build a modular privacy control plane (region-based policy enforcement), and avoid architectural dependence on cross-context behavioral advertising style data sharing. citeturn4search1  

### Safety and welfare risks

**Optimizing for engagement vs welfare**  
A pure engagement objective can promote sensational listings or dogs that are “interesting” but poor fits, increasing returns and welfare harm—especially given evidence that early returns often happen quickly for nuisance behaviors. citeturn1search4turn3search2  

Mitigations:
- multi-objective optimization: explicitly weight retention/welfare proxies (check-ins, reported issues) and constrain recommendations by needs-coverage and risk-alignment gates. citeturn1search4turn0search0  

**Expectation mismatch**  
If the app overpromises (e.g., “perfect fit”) it can worsen expectation mismatch, which is associated with returns. citeturn0search0  

Mitigations:
- calibrated language (“likely fit,” “areas to prepare for”), uncertainty display for behavior traits, and early post-adoption support prompts (training resources, follow-up schedule). citeturn0search0turn1search4turn0search12