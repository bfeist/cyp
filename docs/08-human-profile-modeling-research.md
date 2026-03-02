# Human Profile Modeling Research Plan (Third Pillar)

_Last updated: 2026-03-01_

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

## Human Profiling Framework (v1 → v3)

## v1: Structured onboarding (baseline)

Collect explicit constraints and preferences:

- household composition (children, elders, other pets)
- living context (space, schedule, activity bandwidth)
- budget tolerance (food, grooming, medical, insurance)
- experience level with dogs and training confidence
- acquisition preference (adopt/purchase/either)

## v2: Behavioral inference layer

Collect implicit preference signals:

- which profiles user lingers on
- save/pass patterns
- message initiation patterns
- abandonment points in funnel
- tolerance signals (distance, age, mix-breed openness)

## v3: Adaptive psychometric + outcome learning

Infer deeper style dimensions:

- novelty vs predictability preference
- routine tolerance
- emotional support expectation
- noise/activity tolerance
- training commitment likelihood

Then update profile based on outcomes:

- successful adoption/purchase
- return/re-homing risk signals
- post-match satisfaction
- mismatch reasons

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
