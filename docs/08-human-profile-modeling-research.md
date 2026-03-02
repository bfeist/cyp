# Human Profile Modeling Research Plan (Third Pillar)

_Last updated: 2026-03-02 (confirmed and expanded from Brief 6 deep research)_

> **Research confirmed:** 3-layer profile model validated, trait ontology confirmed, event schema defined (17 event types), MVP model roadmap confirmed, bias/privacy risks documented. See `00-executive-summary.md`, Decision 5.

## Why this matters

Dog matching quality will be capped if user understanding is shallow. A static questionnaire is helpful but insufficient. The third pillar is a **multi-signal human profile system** that continuously improves match quality.

## Goal

Define and validate a profile system that goes beyond self-report and incorporates:

- stated preferences
- behavioral signals
- contextual constraints
- psychometric patterns
- outcome feedback loops

---

## Human Profiling Framework (confirmed v1 → v3 roadmap)

## v1: Structured onboarding + hard constraint gates (confirmed MVP)

Collect explicit constraints and preferences:

- household composition (children, elders, other pets) — **first-class, not a footnote** (pet incompatibility is a top return driver)
- living context (space, schedule, hours alone, activity bandwidth)
- budget tolerance (food, grooming, medical, insurance)
- experience level with dogs and training confidence
- acquisition preference (adopt/purchase/either)
- expectation setting: "what you expect the dog to be like in 2 weeks / 3 months" — treat as editable/monitored, not one-time

**v1 fit score: 5 interpretable weighted components**

| Component               | What it captures                                                               | Evidence basis                                           |
| ----------------------- | ------------------------------------------------------------------------------ | -------------------------------------------------------- |
| Needs coverage          | Exercise/enrichment capacity vs energy/excitability                            | Common return driver: nuisance behavior from unmet needs |
| Risk alignment          | Experience/training willingness vs fearfulness/separation behavior uncertainty | Behavior-related issues are top return category          |
| Household compatibility | Kids/pets/visitors vs dog sociability indicators                               | Pet incompatibility is a top return driver               |
| Care complexity fit     | Grooming/medical routines vs budget/time                                       | Cost of ownership and bond satisfaction                  |
| Bonding style           | Affectionate vs active engagement preference                                   | Dyadic relationship quality                              |

## v2: Behavioral inference layer (ML ranker with bias correction)

Collect implicit preference signals. **Start logging immediately in v1** (missing events = missing training data):

- `dog_view` (must include `position` + `surface` fields for bias correction)
- `dog_save`
- `dog_dismiss` (with optional `dismiss_reason`)
- `message_thread_start`
- `meet_and_greet_scheduled`
- `application_submitted`
- `adoption_completed`
- `post_adoption_checkin` (rating + issues + time_cost_delta)
- `return_reported` (with `reason_code` taxonomy: behavior, pet incompatibility, health/allergies, life circumstances, expectation mismatch)

**Model approach**: learning-to-rank with inverse propensity scoring to correct position bias. Graded labels: view (weak) → save → apply → adopt → retention at 30/90 days.

## v3: Adaptive psychometric + outcome learning

Adaptive elicitation methods (confirmed by research):

- **Pairwise micro-choices**: "Which would you rather meet: Dog A or Dog B?" — Bradley-Terry preference model; active strategy selects maximally informative comparisons
- **Conversational prompts** mapped to ontology nodes: "Describe your weekday routine" → extracts `hours_alone`, `exercise_capacity`, `visitor_frequency`; immediately shown as editable chips
- **Active learning trigger**: ask when posterior uncertainty would materially change recommendations (e.g., user saves high-energy dogs but claimed low-energy)
- **Contextual bandit exploration**: Thompson sampling to surface diverse-but-plausible dogs; improves learning and avoids over-commitment to early noisy signals

---

## Confirmed Trait Ontology (from Brief 6)

### Human-side dimensions (3 layers)

**Layer 1 — Hard constraints** (deterministic gates):
`allergies`, `other_pets`, `housing_type`, `mobility_limits`, `max_hours_alone`, `budget_ceiling`

**Layer 2 — Capacity and lifestyle fit**:
`time_capacity`, `schedule_regularity`, `exercise_capacity`, `training_willingness`, `experience_level`, `tolerance_for_challenging_behavior`, `handling_skill`

**Layer 3 — Preference and bonding style**:
`affectionate_engagement_preference`, `active_engagement_preference`, `novelty_tolerance`, `noise_sensitivity`, `cleanliness_tolerance`, `emotional_support_expectation`, `risk_tolerance_medical_behavioral`

### Dog-side trait groups (store with confidence + provenance + `expires_at`)

- **Temperament/behavior factors**: fearfulness, dog-directed fear, separation behaviors, excitability/energy, trainability/responsiveness, stranger-directed fear, attachment/attention-seeking (mapped to C-BARQ/DPQ factor groupings)
- **Activity and enrichment needs**: energy, play style, stamina, recovery time
- **Sociability**: with dogs, cats, children, visitors
- **Care complexity**: grooming, medical needs, special diets

**Critical design note**: shelter assessments have limited predictive power (especially for aggression); store all behavioral traits probabilistically with multi-source provenance. Never hard-gate on a single shelter assessment.

---

## Confirmed Event Schema (17 event types from Brief 6)

| Event                        | Actor         | Key fields                                                                                           |
| ---------------------------- | ------------- | ---------------------------------------------------------------------------------------------------- |
| `profile_set_constraint`     | user          | `constraint_type`, `value`, `source="explicit"`                                                      |
| `profile_set_preference`     | user          | `preference_type`, `value`, `strength`, `context`                                                    |
| `trait_inferred_update`      | system        | `trait_type`, `value`, `confidence`, `evidence_event_ids`, `model_version`                           |
| `dog_view`                   | user          | `dog_id`, `listing_id`, **`position`** (required), `surface`, `dwell_ms`                             |
| `dog_save`                   | user          | `dog_id`, `listing_id`                                                                               |
| `dog_dismiss`                | user          | `dog_id`, `listing_id`, `dismiss_reason`                                                             |
| `message_thread_start`       | user          | `counterparty_type`, `listing_id`                                                                    |
| `meet_and_greet_scheduled`   | user          | `dog_id`, `datetime`, `location_type`                                                                |
| `application_started`        | user          | `dog_id`, `listing_id`                                                                               |
| `application_submitted`      | user          | `dog_id`, `listing_id`, `self_reported_commitment_score`                                             |
| `adoption_completed`         | partner/admin | `dog_id`, `user_id`, `date`, `channel`, `fee_bucket`                                                 |
| `post_adoption_checkin`      | user          | `days_since_adoption`, `rating`, `issues[]`, `time_cost_delta`                                       |
| `return_reported`            | partner/admin | `dog_id`, `days_since_adoption`, `reason_code`, `free_text`                                          |
| `preference_pairwise_choice` | user          | `option_a_id`, `option_b_id`, `choice`, `question_strategy`                                          |
| `explanation_shown`          | system        | `dog_id`, `explanation_id`, `user_action_after`                                                      |
| `profile_correction`         | user          | `trait_type`, `old_value`, `new_value`, `why_incorrect`                                              |
| `return_reason_taxonomy`     | system        | codes: behavior / pet_incompatibility / health_allergies / life_circumstances / expectation_mismatch |

## Confirmed Bias and Privacy Risk Matrix (from Brief 6)

| Risk                               | Expected manifestation                                              | Confirmed mitigation                                                                                             |
| ---------------------------------- | ------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| Popularity/exposure bias           | Photogenic dogs get all traffic; tail inventory invisible           | Propensity correction (IPS) + exposure quotas + diversity constraints                                            |
| Socioeconomic proxy bias           | Device type / neighborhood as proxy for protected attribute         | Feature governance; fairness testing on exposure and outcomes                                                    |
| Dog labeling bias                  | Overconfident aggression predictions from noisy shelter assessments | Multi-source trait aggregation + uncertainty display + conservative safety gates                                 |
| Overcollection                     | Lifestyle/health data collected beyond stated purpose               | Purpose strings per field, progressive disclosure, short retention for raw logs                                  |
| User profiling rights              | GDPR/CASL automated decision concerns                               | Non-deterministic framing (decision support, not denial); meaningful explanations + correction UX                |
| Engagement vs welfare misalignment | Popular/sensational dogs optimized, not good-fit dogs               | Multi-objective: weight retention/welfare signals; constrain by needs-coverage + risk-alignment gates            |
| Expectation mismatch amplification | App overpromises; returns increase                                  | Calibrated language ("likely fit"); uncertainty display for behavior traits; early post-adoption support prompts |

---

## Confirmed Explanation UX Pattern (from Brief 6)

Explanation format per match:

```
• Constraint match: "Fits your allergy constraint and housing size."
• Lifestyle fit: "Energy level matches your 45–60 min/day walk plan."
• Compatibility: "Marked as comfortable with other dogs; you have a resident dog."
• What to expect (uncertainty-aware): "Some signs of separation stress reported in foster notes;
  recommended: gradual alone-time plan."
```

Profile correction UX (required from day 1):

- Every inferred trait displayed as a chip with confidence: "We think you prefer high-energy dogs (70%)." + one-tap edit
- "Not relevant" and "Stop using this" controls at trait level
- "I dismissed because..." lightweight label capture

---

## Research Workstreams Required

## Workstream A — Trait Taxonomy Definition

Define a practical and modelable trait ontology for humans relevant to dog matching.

Candidate dimensions:

- `activity_capacity`
- `structure_vs_flexibility`
- `cleanliness_grooming_tolerance`
- `noise_sensitivity`
- `training_persistence`
- `social_exposure_preference`
- `risk_tolerance_medical_behavioral`
- `emotional_expectation` (companion vs working/active partner)

Deliverables:

- canonical trait definitions
- observable proxy signals per trait
- mapping to breed/listing features

## Workstream B — Advanced Assessment Methods Beyond Questionnaires

Research and evaluate methods used in modern dating/recommendation systems:

- pairwise preference elicitation (A/B dog profile comparisons)
- forced-choice tradeoff tasks ("lower shedding" vs "higher activity")
- adaptive questioning (next question chosen from uncertainty)
- conversational profiling using LLM-guided prompts
- multi-armed bandit preference probing

Deliverables:

- recommended assessment stack for MVP and phase 2
- signal reliability ranking
- expected completion-friction analysis

## Workstream C — Behavioral Event Instrumentation

Define event taxonomy and data contract.

Required event groups:

- impression events
- click/deep-view events
- save/like/dislike/pass events
- inquiry/contact events
- conversion events (application, adoption, purchase)
- negative outcomes (withdrawn, return, dissatisfaction)

Deliverables:

- event schema
- collection policy
- retention and aggregation rules

## Workstream D — Matching Science & Model Strategy

Compare modeling options:

- rule-based weighted scoring
- gradient boosted tabular rankers
- two-tower embedding retrieval
- hybrid ranking (rules + ML)

Deliverables:

- baseline model design
- feature set specification
- online learning plan from outcomes
- calibration and fairness checks

## Workstream E — Explainability and Trust

User adoption depends on understandable match rationale.

Research questions:

- Which explanation style drives confidence and action?
- How much detail is useful vs overwhelming?
- How should uncertainty be communicated?

Deliverables:

- explanation templates
- confidence expression strategy
- user-facing controls to correct profile assumptions

## Workstream F — Safety, Bias, and Ethical Risk

Assess risk areas:

- socioeconomic bias from budget/location features
- household stereotype bias
- overconfident temperament predictions
- penalizing mixed/unknown profiles

Deliverables:

- bias test matrix
- mitigation strategies
- governance checkpoints before model promotion

---

## Data Requirements for Human Profiling

## Minimum data objects

- `HumanProfile` (explicit constraints)
- `HumanSignalEvent` (behavior stream)
- `PreferenceEmbedding` (latent representation)
- `MatchOutcome` (short and long horizon outcomes)
- `ProfileRevision` (auditable profile state changes)

## Label strategy

Short-term labels:

- click-through
- save
- inquiry
- start application

Long-term labels:

- successful adoption/purchase
- retention at 30/90/180 days
- owner satisfaction
- return/rehome events

---

## Research Experiments to Run

## Experiment 1 — Questionnaire-only vs hybrid profile

Hypothesis: hybrid (explicit + behavior) yields significantly higher conversion and satisfaction.

## Experiment 2 — Adaptive questioning vs fixed flow

Hypothesis: adaptive questioning reduces abandonment and improves profile confidence.

## Experiment 3 — Explanation format A/B

Hypothesis: concise factor-based explanation outperforms opaque score-only UI.

## Experiment 4 — Drift-aware profile updates

Hypothesis: controlled profile drift updates reduce mismatch over time.

---

## Metrics for Success

## Matching quality metrics

- inquiry rate per recommendation impression
- adoption/purchase conversion rate
- time-to-match
- return/rehome rate
- post-match satisfaction score

## Modeling metrics

- NDCG@k / MAP@k for ranking
- calibration error of compatibility probabilities
- uplift vs rule baseline
- stability under profile drift

## Responsible AI metrics

- group-level performance parity checks
- false-confidence rate
- explanation usefulness rating

---

## External Deep Research Sessions You Should Run

## Session A — Dating-app profiling advances transferable to pet matching

Request:

- latest approaches in preference learning, psychometrics, and behavioral ranking
- methods with strong evidence for reducing mismatch
- practical implementation patterns for startups

## Session B — Human-animal bond outcome predictors

Request:

- literature on factors predicting successful dog-owner fit and retention
- validated scales or instruments relevant to commitment, training adherence, and lifestyle fit

## Session C — Responsible profiling and fairness

Request:

- best practices for adaptive profiling systems that avoid discriminatory outcomes
- explainability patterns for consumer recommendation systems

---

## Implementation Recommendation

For MVP, use a **hybrid profile**:

1. compact onboarding questionnaire
2. pairwise preference mini-tasks
3. behavioral event capture from browsing/messaging
4. transparent score explanation with correction controls

Then iterate with outcome-linked model updates instead of adding long static forms.
