# Designing an NLP Extraction System for Shelter and Rescue Dog Temperament Attributes

## Executive summary

Shelter/rescue dog descriptions are typically free-text “marketing + field notes” hybrids: they mix upbeat adjectives, compatibility claims (“good with dogs”), training status (“house-trained”), and sometimes safety constraints (“no cats”), often written by different people and updated over time. Because a dog’s observed behavior can vary strongly by context (kennel vs. foster home vs. outdoors) and because single temperament tests/behavior assessments have limited predictive value—especially for rare but high-stakes behaviors like post-adoption aggression—an extraction system should **avoid treating every sentence as equally reliable** and must preserve **provenance, context, and uncertainty** rather than outputting a single brittle label. citeturn13view0turn14view0turn14view1

This report proposes (a) a hierarchical ontology centered on **reactivity, sociability, prey drive, energy, separation behavior, and trainability cues**, aligned where possible with validated behavioral constructs (e.g., standardized questionnaire subscales and their definitions/scales), and (b) a hybrid extraction pipeline that combines **high-recall candidate detection** (lexicon/pattern rules) with **supervised sequence labeling/classification**, **negation/speculation/context tagging**, **multi-sentence aggregation**, **probability calibration**, and **contradiction detection** that distinguishes true inconsistency from “different context” or “behavior changed over time.” citeturn12view0turn26view0turn1search0turn1search5turn1search2turn4search1

Key implementation recommendations:

- Represent each extracted attribute as a set of **evidence-grounded assertions** (value + polarity + certainty + time/context + source + calibrated confidence), not as a single “final” trait. This mirrors proven clinical-NLP ideas (negation/hypothetical/historical/experiencer properties) and reduces downstream harm from overconfident summaries. citeturn1search1turn1search0turn13view0  
- Assign confidence using **calibrated probabilities** (e.g., temperature scaling) and optionally **epistemic uncertainty** (e.g., deep ensembles or MC dropout), then aggregate to attribute-level confidence with explicit handling of contradictory evidence. citeturn1search6turn2search2turn2search5turn21search0  
- Detect contradictions via a two-tier approach: (1) fast ontology/rule checks (mutually exclusive values, polarity flips under same context/time), then (2) a learned textual inconsistency layer using Natural Language Inference (NLI) models and/or document-level NLI for multi-sentence conflicts. citeturn1search3turn4search1turn4search0  
- Anticipate biased language: many bio-writing resources explicitly encourage positive framing and even recommend omitting “stop sign” negatives from public marketing copy, so absence of mention is not evidence of absence, and euphemisms are common. citeturn18view0turn9view0turn10view1  

## Ontology draft

### Design principles

A practical ontology for shelter/rescue descriptions should satisfy four constraints:

1. **Behavior is conditional**: store **stimulus + context** (e.g., “reactive to dogs *on leash*” vs “plays well off-leash”), reflecting that reactivity can appear only in particular contexts. citeturn23view0turn13view0  
2. **Behavior changes over time**: capture temporal cues (“used to…”, “now…”, “after decompressing…”) because post-adoption and post-settling changes are documented. citeturn16view1turn14view0  
3. **Sources differ in reliability**: weigh statements differently (previous owner vs staff vs foster home vs standardized test), and retain provenance to support auditing and human review. citeturn13view0turn9view1  
4. **Marketing language is not neutral measurement**: many listings emphasize adjectives (e.g., “couch potato”, “energetic”, “social butterfly”) and may avoid “barrier” phrases; build separate fields for *observed behavior* vs *promotional descriptors* and avoid over-inference. citeturn9view0turn18view0turn20view0turn10view1  

### Hierarchical label tree

Below is a recommended **medium-granularity** hierarchy (two levels deep) that satisfies the requested dimensions while staying annotatable.

```yaml
temperament_behavior:
  reactivity:
    target_stimulus: [dogs, strangers, children, handling, noises_objects, vehicles, other]
    context: [on_leash, behind_barrier, in_kennel, in_home, off_leash, during_handling, unknown]
    valence: [reactive_display, fearful_avoidant, frustrated_overaroused, calm_neutral, unknown]
    intensity: [none, mild, moderate, high, unknown]
  sociability:
    with_adults: [low, medium, high, unknown]
    with_children: [no, supervised_only, yes, unknown]
    with_dogs: [no, selective, yes, unknown]
    with_cats: [no, supervised_only, yes, unknown]
  prey_drive:
    chasing_small_animals: [low, moderate, high, unknown]
    cat_safety_proxy: [unsafe, uncertain, likely_safe, unknown]
  energy:
    energy_level: [low, medium, high, unknown]
    exercise_need_minutes_per_day: numeric_or_unknown
    settle_ability: [settles_easily, settles_with_help, difficult_to_settle, unknown]
  separation_behavior:
    separation_distress: [none, mild, moderate, severe, unknown]
    confinement_crate_distress: [none, mild, moderate, severe, unknown]
  trainability_cues:
    command_knowledge: [none_reported, basic_cues, advanced_cues, unknown]
    housetraining_status: [trained, in_progress, not_trained, unknown]
    leash_skills: [loose_leash, pulls, unknown]
    motivation: [food_motivated, toy_motivated, praise_motivated, mixed, unknown]
  unspecified_other:
    other_behavioral_notes: free_text_spans
    medical_or_contextual_behavior_modifiers: free_text_spans
```

This structure is intentionally centered on **(attribute, stimulus/target, context, intensity)**, because the same dog can be calm in one environment and reactive in another, and shelters are explicitly encouraged to gather multiple sources and contexts rather than rely on any single observation. citeturn13view0turn23view0  

### Label specifications (definitions, cues, value types, granularity)

The table below gives concrete label definitions and surface cues. Cues are grounded in common shelter marketing/adjective lists and observed recurring listing phrases (e.g., “couch potato”, “energetic”, “only dog”, “no cats”) and in standardized construct definitions where available. citeturn9view0turn10view1turn12view0  

| Label (path) | Definition | Typical lexical cues / phrases (non-exhaustive) | Expected value type | Recommended granularity |
|---|---|---|---|---|
| reactivity.target_stimulus | Who/what triggers disproportionate arousal or reactive responses | “reactive to dogs”, “barks at strangers”, “startles at loud noises”, “fearful of traffic”, “handling sensitive” citeturn23view0turn12view0turn23view1 | Categorical (one-of set) + multi-label | Medium: capture target classes; avoid ultra-fine triggers unless you can annotate reliably |
| reactivity.context | Environment/constraint where reactivity appears | “on leash”, “behind a fence”, “in kennel”, “outside”, “in the home”, “at the vet” citeturn23view0turn13view0 | Categorical + multi-label | Medium: on_leash vs off_leash vs barrier vs handling are high-yield |
| reactivity.valence | Behavioral style of reactivity | “lunges/barks/growls”, “cowers”, “freezes”, “overexcited”, “frustrated”, “fearful” citeturn23view0turn24view0turn23view1 | Categorical | Coarse-to-medium: keep 4–5 bins; finer subtypes tend to reduce agreement |
| reactivity.intensity | Severity estimate under stated context | “a little”, “sometimes”, “often”, “strongly”, “goes nuts”, “manageable” | Ordinal (none/mild/moderate/high/unknown) | Medium: 4-level ordinal works better than boolean for triage |
| sociability.with_adults | Affiliation/comfort with adults | “people person”, “loves everyone”, “shy at first”, “warms up”, “independent” citeturn9view0turn12view0turn23view1 | Ordinal / categorical | Medium: low/med/high + unknown; optionally add “shy_then_warms” as modifier |
| sociability.with_children | Compatibility or constraints with kids | “kid-friendly”, “older kids only”, “no kids”, “gentle with children” citeturn10view1turn20view0 | Categorical | Medium: (no / supervised_only / yes / unknown) beats age-threshold micro-labels |
| sociability.with_dogs | Comfort/play compatibility with dogs | “dog friendly”, “plays well”, “selective”, “prefers to be the only dog”, “needs slow intros” citeturn12view0turn10view1turn13view0 | Categorical | Medium: no / selective / yes / unknown; add context flags (leash vs off leash) |
| sociability.with_cats | Cat compatibility stated | “cat-friendly”, “no cats”, “chases cats”, “unknown with cats” citeturn10view1turn12view0 | Categorical | Medium; tie into prey-drive contradictions (below) |
| prey_drive.chasing_small_animals | Predatory chasing propensity (proxy for prey drive) | “high prey drive”, “chases squirrels”, “will chase cats/birds”, “strong chase instinct” citeturn12view0turn23view1 | Ordinal (low/moderate/high/unknown) | Medium; avoid too many sub-stages of predation in shelter context |
| prey_drive.cat_safety_proxy | Derived “cat safety” risk indicator from prey-drive statements | “no cats”, “cat test failed”, “must be only pet”, “fixates” | Categorical (unsafe/uncertain/likely_safe/unknown) | Medium; treat as *derived* with explanation |
| energy.energy_level | General energy/activity level | “low-key”, “couch potato”, “chill”, “energetic”, “busy bee”, “always on the go” citeturn9view0turn12view0 | Ordinal (low/med/high/unknown) | Medium; can map adjective lists into bins |
| energy.exercise_need_minutes_per_day | Claimed daily exercise dose | “needs 2 long walks”, “30–60 minutes”, “runs” | Numeric (minutes/day) or unknown | Optional numeric; only fill when explicit quantities appear |
| energy.settle_ability | Ability to calm/settle after arousal | “settles nicely”, “has difficulty settling”, “crate helps settle” citeturn12view0turn16view1 | Categorical | Medium; important for adopter fit and explains “high energy” nuance |
| separation_behavior.separation_distress | Distress when left alone (separation-related behaviors) | “separation anxiety”, “panics when alone”, “destructive when left”, “barks/howls when alone”, “escape attempts” citeturn24view0turn24view1turn12view0 | Ordinal (none/mild/moderate/severe/unknown) | Medium; treat “possible” vs “diagnosed” via certainty tagging |
| separation_behavior.confinement_crate_distress | Distress when confined (crate/kennel) | “crate anxiety”, “kennel stress”, “barrier frustration”, “breaks out of crate” citeturn13view0turn24view1 | Ordinal | Medium; distinguish from separation (alone) where possible |
| trainability_cues.command_knowledge | Reported known cues / responsiveness to instruction | “knows sit/down”, “responds to ‘come’”, “learns quickly”, “well-trained” citeturn12view0turn13view0 | Categorical | Medium; keep “basic vs advanced” rather than enumerating every cue |
| trainability_cues.housetraining_status | Household elimination training status | “house-trained”, “potty trained”, “has accidents”, “still learning” citeturn24view0turn12view0 | Categorical | Medium; important adopter-fit attribute; requires negation handling (“not house-trained”) |
| trainability_cues.leash_skills | Leash manners | “loose-leash”, “pulls”, “needs leash training” citeturn13view0turn23view0 | Categorical | Medium; include “on-leash reactivity” separately under reactivity |
| trainability_cues.motivation | Reinforcer preference hints | “food motivated”, “loves treats”, “toy-driven” | Categorical | Coarse-to-medium; high uncertainty unless explicitly described |
| unspecified_other.other_behavioral_notes | Catch-all for meaningful behaviors not covered | “noise phobia”, “guarding”, “car anxious”, etc. citeturn12view0turn13view0 | Free-text spans + optional classifier | Keep; prevents forced mislabeling and supports iterative ontology expansion |
| unspecified_other.medical_or_contextual_behavior_modifiers | Medical/pain/stress qualifiers affecting behavior | “in pain”, “post-surgery”, “kennel stress”, “decompression” citeturn14view0turn13view0 | Free-text spans + optional tags | Keep; critical for safe interpretation |

### Label-set variants and trade-offs

| Variant | Approx. leaf labels | What you gain | What you lose | Best fit scenarios |
|---|---:|---|---|---|
| Coarse | 6–10 | Fast annotation, higher inter-annotator agreement, simpler models | Low specificity; weak contradiction handling (too many statements collapse into same bin) | Early MVP; search filters (“high energy”, “dog-friendly”) |
| Medium (recommended) | 20–35 | Captures context (leash vs off-leash), compatibility, and key welfare risks; contradiction detection becomes meaningful | Moderate annotation cost; model complexity increases | Most shelters/rescues; decision support + QA review |
| Fine | 50–100+ | Detailed adopter matching and counseling support; supports nuanced behavioral plans | Lower agreement; requires large labeled corpora; contradictions may increase due to over-fragmentation | Large multi-shelter networks; research-grade analytics |

Coarse ontologies tend to produce overconfident summaries that conflict with guidance to weigh multiple information sources and contexts. Medium granularity better reflects how shelters are advised to integrate information across settings and time. citeturn13view0turn16view1  

## Pipeline architecture

### End-to-end flow (Mermaid)

```mermaid
flowchart TD
  A[Ingest text + metadata\n(listing, kennel card, foster notes,\nowner surrender form, behavior test notes)] --> B[Preprocess\nnormalize, de-dup, sentence split,\nsection detect, spelling variants]
  B --> C[Candidate span detection\nlexicons + regex + rule matchers\n(high recall)]
  C --> D[Neural IE models\nsequence labeling (NER) + attribute classifiers\n+ relation extraction (attribute↔target↔context)]
  D --> E[Context tagging\nnegation + speculation + experiencer/source\n+ temporal cues + environment cues]
  E --> F[Coreference + entity linking\n(dog, other animals, pronouns)]
  F --> G[Multi-sentence aggregation\nmerge evidence, compute attribute values\nper context/time/source]
  G --> H[Confidence estimation\ncalibrate probabilities + uncertainty signals\n(token→span→attribute)]
  H --> I[Contradiction detection\nontology rules + NLI-based checks\n+ resolution policy]
  I --> J[Structured output\nattributes + confidences + evidence + conflicts]
  J --> K[Human-in-the-loop review\nqueue low-confidence or conflicts]
  K --> L[Feedback loop\nactive learning + guideline refinement]
  L --> D
```

This architecture explicitly separates (1) “find relevant spans” from (2) “interpret them in context” and (3) “decide how confident we are and whether statements conflict,” reflecting proven patterns in negation/context-aware extraction. citeturn1search0turn1search1turn7search0turn7search3  

### Data collection and preprocessing

**Data sources (recommended):** public adoption listings; internal kennel cards; staff daily notes; foster notes; standardized questionnaires at intake; behavior assessment summaries; adopter follow-up notes. Shelters are advised to treat each as “a piece of the puzzle,” and to avoid decisions based on a single context. citeturn13view0turn14view0  

**Preprocessing steps:**
- **Section and template detection:** Many bios follow semi-structured patterns (“Likes/Dislikes”, “Ideal home”, “Needs”). Section-aware models reduce false relations (e.g., don’t treat “Needs: fenced yard” as behavior). citeturn20view0  
- **Normalization:** expand shorthand (“w/”, “w/o”), handle breed nicknames, normalize negation variants (“doesn’t”, “wont”, “won’t”). Negation errors are a top cause of extraction mistakes in multiple domains. citeturn1search0turn1search27  
- **De-dup and revision tracking:** listings are often edited; store versions for temporal reasoning and to detect “contradictions” that are actually updates. citeturn16view1turn13view0  

### Candidate span detection (high recall front-end)

A robust system should prioritize **recall** early, then rely on later classifiers to reduce false positives.

**Techniques to consider:**
- **Lexicon + phrase tables** for common descriptors (energy/activity adjectives, sociability descriptors, compatibility phrases). Public adjective lists for bios provide a starting lexicon but should be treated as *marketing language* rather than measurement. citeturn9view0turn10view1  
- **Regex/pattern rules** for canonical constraints: “no cats”, “only dog”, “older kids”, “must be leashed”, “cannot live with”. These are high-precision triggers and are explicitly common in profile corpora. citeturn10view1turn13view0  
- **Token-pattern matchers** (rule engines that operate over token attributes, not just raw strings) for better robustness than regex alone. citeturn6search2  

Output of this stage: **candidate spans** with tentative label hypotheses + offsets. Keep all candidates and assign low initial confidence rather than dropping uncertain cases.

### Model layer: NER / classification / relation extraction

Because many behavioral statements are short and phrase-like (“dog selective”, “gentle giant”), the task is a blend of **sequence labeling (NER)** and **attribute classification**:

1. **Sequence labeling / NER (span detection + typing):**
   - Baseline: CRF with lexical + character n-grams + POS/dependency features; CRFs remain a strong baseline for span labeling. citeturn6search0  
   - Modern: transformer encoder fine-tuning for token classification or span-based NER. citeturn6search1turn4search2  

2. **Attribute classification (sentence- or span-level):**
   - Multi-label classification for targets (“dogs”, “cats”, “strangers”) and contexts (“on leash”, “in kennel”). Contextual reactivity is explicitly recognized as conditional (e.g., on-leash only). citeturn23view0  
   - Ordinal classification (energy intensity, separation severity). Ordinal scales map naturally to validated questionnaire scoring conventions (0–4 severity/frequency). citeturn26view0turn26view1  

3. **Relation extraction:**
   - Needed when one sentence contains multiple traits and targets (“loves people but is selective with dogs”).  
   - Options: dependency-pattern rules; or joint entity–relation extraction using span-based transformer models to predict entity pairs/relations. citeturn6search3turn4search3  

**Multi-task learning:** jointly training NER + context tags + attribute heads can reduce pipeline error propagation and improve label consistency across correlated traits (e.g., reactivity target vs sociability). citeturn6search3turn4search3  

**Prompt-based LLMs:** useful (a) as a rapid bootstrap annotator for weak labels and (b) as a *rationale generator* for human review, but should not be the sole extractor unless you can enforce calibration and evidence grounding; modern neural systems are often miscalibrated without explicit calibration. citeturn1search2turn1search6  

### Negation, speculation, and “who observed it” (context properties)

Shelter text frequently includes: negation (“not good with cats”), uncertainty (“may be shy at first”), and provenance (“foster reports…”). Treat these as first-class fields:

- **Negation detection:** start with lexicon + rule scope (NegEx-style) then extend with contextual triggers and/or neural scope models when coverage demands. citeturn1search0turn1search1turn6search14  
- **Speculation/hedging:** detect “may/might/seems/likely”, “still learning”; biomedical NLP provides established approaches and corpora for speculation cues and scope. citeturn7search0turn7search1  
- **Experiencer/source analog:** in shelter text, “experiencer” generalizes to **observer/source** (previous owner vs staff vs foster). Context-aware NLP demonstrates how to tag whether a condition is negated, hypothetical, or historical; adopt the same pattern. citeturn1search1turn13view0  

### Temporal and context resolution

**Temporal cues** matter because behavior can change with “settling in,” and longitudinal work shows some subscales can change over months after adoption (e.g., changes in excitability, chasing, separation-related behaviors). citeturn16view1turn16view3  

Recommended approach:
- Rule-based time expression normalization (e.g., “after 2 weeks”) and simple “phase” tagging (intake / in-shelter / foster / post-adoption). citeturn7search2  
- Detect discourse markers: “at first… now…”, “used to…”, “has been improving”.  
- Store attributes as **time-scoped** entries rather than overwriting.

### Coreference and cross-sentence aggregation

Listings often use pronouns and implicit subjects. Coreference models can cluster mentions and help attribute statements correctly (especially in multi-dog fosters or “bonded pair” listings). citeturn7search3  

Aggregation strategy:
- Build an **evidence graph** where each node is an extracted assertion (attribute, value, polarity, certainty, context, time, source, span).  
- Collapse nodes into **attribute views** per context/time (e.g., sociability.with_dogs in kennel vs in foster).  
- When multiple pieces of evidence support the same value, increase confidence via weighted pooling; when evidence conflicts, invoke contradiction logic.

### Confidence scoring and calibration

**Why calibration matters:** modern neural networks can be systematically overconfident; temperature scaling is an effective, simple post-hoc calibration method on held-out data, and ECE is a common scalar measure of miscalibration. citeturn1search2turn1search6  

Recommended confidence design (three layers):

1. **Token/span confidence:** model probability for the span label and for polarity/certainty tags.
2. **Assertion confidence:** combine span probability with (a) negation/speculation penalties and (b) source weighting (e.g., foster-home observations may better reflect home behavior than kennel-only observations, consistent with guidance). citeturn13view0turn14view0  
3. **Attribute-level confidence:** aggregate multiple assertions per attribute. Prefer producing:
   - `p(value = v | evidence)` over the allowed discrete set, not just a single score.
   - `confidence` + `coverage` decisions (abstain/needs_review if below threshold), consistent with selective prediction framing. citeturn21search7turn21search11  

**Uncertainty estimation options:**
- **Deep ensembles** for epistemic uncertainty via disagreement across independently trained models. citeturn2search5turn2search1  
- **MC dropout** as an approximate Bayesian approach to uncertainty. citeturn2search2turn2search6  
- Distinguish **aleatoric vs epistemic** uncertainty conceptually to guide workflow (e.g., epistemic → more labeling helps; aleatoric → inherent ambiguity). citeturn21search0  

### Contradiction detection and resolution

A key risk in dog profiles is conflicting claims (“good with cats” vs “chases cats”). Contradictions can arise from:
- Different contexts (on leash vs off leash) mistaken as inconsistency. citeturn23view0  
- Updates over time (“was shy at first”). citeturn16view1  
- Biased reporting and limited predictive validity of single tests. citeturn13view0turn14view1  

**Two-tier contradiction detection:**

1. **Ontology/rule-based checks (fast, explainable):**
   - Polarity flip within same context/time: “not reactive to dogs” vs “reactive to dogs” (same context).  
   - Mutually exclusive bins: sociability.with_dogs = yes vs no (same time/context).  
   - Derived conflicts: prey_drive high + “cat-friendly” without qualifiers → flag as potential contradiction rather than hard contradiction.

2. **Learned inconsistency checks (NLI):**
   - Convert each assertion into a canonical hypothesis (“The dog is dog-friendly.” “The dog is not dog-friendly.”), then run NLI between evidence sentences and hypotheses. Large NLI corpora and document-level NLI support this paradigm. citeturn1search3turn4search1  
   - For “listing-wide consistency,” treat the full listing (or section) as premise and each hypothesis as query (document-level NLI). citeturn4search1turn4search13  

**Resolution policy (deterministic + auditable):**
- Prefer higher-confidence, higher-reliability sources; require corroboration for severe claims when guidance recommends multiple sources before high-stakes decisions. citeturn13view0  
- Prefer **newer** statements **only if** they are not speculative and have comparable source reliability.  
- If conflict remains: output `status = unresolved_contradiction` and route to human review with evidence spans.

### Training data, annotation schema, and active learning

**Training needs:** medium ontology reliably typically requires thousands of labeled mentions (not just documents), because many traits (e.g., severe separation distress) are comparatively rare in text or euphemized. The adoption-marketing practice of omitting “stop signs” implies strong class imbalance and missing not at random (MNAR) patterns. citeturn18view0turn13view0  

**Annotation schema (recommended):**
- **Span labels:** trait mention spans (e.g., “couch potato”, “dog selective”).  
- **Attributes:** normalized label path + value.  
- **Context properties:** negation, speculation, time-scope, context (on leash/kennel/home), source (owner/foster/staff/test). citeturn1search1turn7search0turn13view0  

**Bootstrapping and weak supervision:**
- Use rules/lexicons as labeling functions to create noisy training labels, then denoise/learn an end model; data programming systems formalize this approach. citeturn2search3turn2search7  

**Active learning strategy:**
- Uncertainty sampling + disagreement sampling: select examples where (a) calibrated confidence is near decision thresholds or (b) ensemble members disagree most. citeturn2search0turn2search5  

### Deployment considerations

- **Latency:** hybrid pipelines can be optimized by running rule-based candidates first and only invoking heavier models when triggers appear; this pattern is common for production IE. citeturn6search2turn6search1  
- **Explainability:** always store evidence spans and applied normalization rules; contradiction flags should cite the conflicting evidence. This mitigates the known problem that behavior assessments and descriptions can be misinterpreted or over-trusted. citeturn13view0turn14view1  
- **Privacy:** treat shelter notes and owner surrender forms as sensitive; apply redaction for personal identifiers before model training and enforce access controls in downstream search/UI (organizational policy requirement; not specific to a single source). citeturn13view0  

## Example input/output pairs

The examples below are synthetic but designed to reflect real listing patterns (adjective-heavy bios, compatibility constraints, and context qualifiers). Confidence values illustrate calibrated outputs; contradictions are explicitly marked.

### Example

**Input listing text**
> “Biscuit is a lovable couch potato who enjoys short strolls and long naps. She’s polite with visitors and loves to snuggle. In the shelter she sometimes barks at other dogs when walking past kennels, but in playgroup she was relaxed and social.”

**Structured output**
```json
{
  "dog_id": "ex1_biscuit",
  "attributes": {
    "energy.energy_level": { "value": "low", "confidence": 0.86, "evidence": ["lovable couch potato", "enjoys short strolls"] },
    "sociability.with_adults": { "value": "high", "confidence": 0.78, "evidence": ["polite with visitors", "loves to snuggle"] },
    "reactivity.target_stimulus": { "value": ["dogs"], "confidence": 0.72, "evidence": ["barks at other dogs"] },
    "reactivity.context": { "value": ["in_kennel", "behind_barrier"], "confidence": 0.66, "evidence": ["walking past kennels"] },
    "reactivity.intensity": { "value": "mild", "confidence": 0.58, "evidence": ["sometimes barks"] },
    "sociability.with_dogs": { "value": "yes", "confidence": 0.69, "evidence": ["in playgroup... relaxed and social"], "context": "off_leash" }
  },
  "contradictions": [
    {
      "attribute": "sociability.with_dogs",
      "type": "contextual_difference_not_true_contradiction",
      "evidence_a": "sometimes barks at other dogs when walking past kennels",
      "evidence_b": "in playgroup she was relaxed and social",
      "note": "Different contexts (barrier/on-leash vs off-leash) commonly yield different reactions."
    }
  ]
}
```

### Example

**Input listing text**
> “Ranger is smart and eager to learn—already knows sit, down, and ‘touch’. He’s friendly with adults, but can be shy at first. He’s *not* good with cats (will chase).”

**Structured output**
```json
{
  "dog_id": "ex2_ranger",
  "attributes": {
    "trainability_cues.command_knowledge": { "value": "basic_cues", "confidence": 0.92, "evidence": ["knows sit, down, and 'touch'"] },
    "sociability.with_adults": { "value": "medium", "confidence": 0.66, "evidence": ["friendly with adults", "shy at first"], "modifier": "warms_up" },
    "sociability.with_cats": { "value": "no", "confidence": 0.94, "evidence": ["not good with cats"] },
    "prey_drive.chasing_small_animals": { "value": "high", "confidence": 0.87, "evidence": ["will chase"] },
    "prey_drive.cat_safety_proxy": { "value": "unsafe", "confidence": 0.90, "evidence": ["not good with cats", "will chase"] }
  },
  "contradictions": []
}
```

### Example

**Input listing text**
> “Luna is an energetic, playful girl who would love a running buddy. She is dog-friendly off-leash, but becomes leash-reactive in crowded areas. She is improving with training.”

**Structured output**
```json
{
  "dog_id": "ex3_luna",
  "attributes": {
    "energy.energy_level": { "value": "high", "confidence": 0.88, "evidence": ["energetic", "playful", "running buddy"] },
    "sociability.with_dogs": { "value": "yes", "confidence": 0.74, "evidence": ["dog-friendly off-leash"], "context": "off_leash" },
    "reactivity.target_stimulus": { "value": ["dogs"], "confidence": 0.77, "evidence": ["becomes leash-reactive"] },
    "reactivity.context": { "value": ["on_leash"], "confidence": 0.83, "evidence": ["leash-reactive"] },
    "reactivity.intensity": { "value": "moderate", "confidence": 0.55, "evidence": ["in crowded areas"] },
    "unspecified_other.other_behavioral_notes": { "value": ["improving with training"], "confidence": 0.52 }
  },
  "contradictions": [
    {
      "attribute": "sociability.with_dogs",
      "type": "contextual_difference_not_true_contradiction",
      "evidence_a": "dog-friendly off-leash",
      "evidence_b": "becomes leash-reactive",
      "note": "Store both: sociability-offleash and reactivity-onleash."
    }
  ]
}
```

### Example

**Input listing text**
> “Milo is affectionate once he trusts you. He may be nervous around strangers and could bark if startled. In foster, he settled quickly and enjoyed having a quiet place to nap.”

**Structured output**
```json
{
  "dog_id": "ex4_milo",
  "attributes": {
    "sociability.with_adults": { "value": "medium", "confidence": 0.60, "evidence": ["affectionate once he trusts you"], "modifier": "warms_up" },
    "reactivity.target_stimulus": { "value": ["strangers", "noises_objects"], "confidence": 0.58, "evidence": ["nervous around strangers", "bark if startled"] },
    "reactivity.valence": { "value": "fearful_avoidant", "confidence": 0.49, "evidence": ["nervous", "startled"] },
    "reactivity.intensity": { "value": "mild", "confidence": 0.44, "evidence": ["could bark"] },
    "energy.settle_ability": { "value": "settles_easily", "confidence": 0.71, "evidence": ["settled quickly", "quiet place to nap"], "context": "in_home" }
  },
  "contradictions": []
}
```

### Example

**Input listing text**
> “Daisy is a social butterfly with people and loves kids. She would prefer to be the only dog in the home. However, staff note she has done well in group play with calm dogs.”

**Structured output**
```json
{
  "dog_id": "ex5_daisy",
  "attributes": {
    "sociability.with_adults": { "value": "high", "confidence": 0.84, "evidence": ["social butterfly with people"] },
    "sociability.with_children": { "value": "yes", "confidence": 0.80, "evidence": ["loves kids"] },
    "sociability.with_dogs": {
      "value": "selective",
      "confidence": 0.62,
      "evidence": ["prefer to be the only dog", "done well in group play with calm dogs"],
      "notes": "Preference statement + partial positive evidence suggests selectivity not absolute incompatibility."
    }
  },
  "contradictions": [
    {
      "attribute": "sociability.with_dogs",
      "type": "soft_conflict",
      "evidence_a": "prefer to be the only dog",
      "evidence_b": "done well in group play with calm dogs",
      "note": "Flag for counseling; may reflect home-management preference rather than aggression."
    }
  ]
}
```

### Example

**Input listing text**
> “Atlas is crate-trained and house-trained. When left alone he has been calm—no destructive behavior reported. He’s high energy and needs daily enrichment.”

**Structured output**
```json
{
  "dog_id": "ex6_atlas",
  "attributes": {
    "trainability_cues.housetraining_status": { "value": "trained", "confidence": 0.90, "evidence": ["house-trained"] },
    "trainability_cues.command_knowledge": { "value": "unknown", "confidence": 0.30, "evidence": [] },
    "separation_behavior.separation_distress": { "value": "none", "confidence": 0.63, "evidence": ["has been calm", "no destructive behavior reported"], "certainty": "reported_observation" },
    "separation_behavior.confinement_crate_distress": { "value": "none", "confidence": 0.70, "evidence": ["crate-trained"] },
    "energy.energy_level": { "value": "high", "confidence": 0.86, "evidence": ["high energy", "needs daily enrichment"] }
  },
  "contradictions": []
}
```

### Example

**Input listing text**
> “Pepper is sweet and silly. She *can’t live with cats*. Another note says she ‘ignores cats in the yard’ but ‘may chase if they run.’”

**Structured output**
```json
{
  "dog_id": "ex7_pepper",
  "attributes": {
    "sociability.with_cats": { "value": "no", "confidence": 0.84, "evidence": ["can't live with cats"] },
    "prey_drive.chasing_small_animals": { "value": "moderate", "confidence": 0.58, "evidence": ["may chase if they run"] },
    "prey_drive.cat_safety_proxy": { "value": "unsafe", "confidence": 0.77, "evidence": ["can't live with cats", "may chase if they run"] }
  },
  "contradictions": [
    {
      "attribute": "sociability.with_cats",
      "type": "explicit_contradiction_requires_review",
      "evidence_a": "can't live with cats",
      "evidence_b": "ignores cats in the yard",
      "note": "Resolve via context + policy: 'ignores' may be conditional; 'can't live with' is a strong placement constraint."
    }
  ]
}
```

### Example

**Input listing text**
> “Shadow is calm in the home and loves to relax. On walks, he barks and lunges at cyclists and sometimes at men wearing hats. He’s working on it and has improved.”

**Structured output**
```json
{
  "dog_id": "ex8_shadow",
  "attributes": {
    "energy.energy_level": { "value": "low", "confidence": 0.66, "evidence": ["calm in the home", "loves to relax"], "context": "in_home" },
    "reactivity.target_stimulus": { "value": ["vehicles", "strangers"], "confidence": 0.75, "evidence": ["barks and lunges at cyclists", "men wearing hats"] },
    "reactivity.context": { "value": ["on_leash"], "confidence": 0.72, "evidence": ["on walks"] },
    "reactivity.intensity": { "value": "moderate", "confidence": 0.57, "evidence": ["barks and lunges"] },
    "unspecified_other.other_behavioral_notes": { "value": ["has improved", "working on it"], "confidence": 0.55, "certainty": "trend_statement" }
  },
  "contradictions": [
    {
      "attribute": "energy.energy_level",
      "type": "no_contradiction_context_split",
      "evidence_a": "calm in the home",
      "evidence_b": "barks and lunges at cyclists",
      "note": "Energy vs reactivity are different constructs; keep context-scoped attributes."
    }
  ]
}
```

## Evaluation plan

### Annotation guidelines

**Unit of annotation:** one listing document (or one dated note) segmented into sentences; annotate both **spans** and **normalized attributes**.

**What annotators must label (minimum):**
- (A) **Trait span**: exact text span.  
- (B) **Normalized label path + value** (from ontology).  
- (C) **Context properties**: negation, speculation, time-scope, context (on leash/kennel/home), and source/observer when stated. Context tagging is critical because shelters explicitly recommend integrating multiple sources and recognizing that behavior varies across environments. citeturn13view0turn1search1turn7search0  

**Negation/speculation rules (examples):**
- “*Not* good with cats” → sociability.with_cats = no, polarity=negated? (store as strong negative placement constraint; not “absence of mention”). citeturn1search0turn13view0  
- “May be shy at first” → sociability.with_adults = low/medium with certainty=speculative. citeturn7search0turn7search1  
- “Was reactive at intake, improving now” → two time-scoped assertions, not a contradiction. citeturn16view1turn14view0  

**Edge cases to standardize:**
- Promotional adjectives with weak behavioral specificity (“sweet”, “love bug”): annotate only if guideline defines mapping; otherwise store under unspecified_other. Bio adjective lists show many descriptors are marketing oriented and not tightly behavior-anchored. citeturn9view0turn10view1  
- “Only dog” vs “selective with dogs”: treat “only dog” as home-placement constraint (often not a claim of universal dog aggression) unless explicitly linked to negative behavior. citeturn10view1turn13view0  
- “House-trained” vs “no accidents in foster”: both support house-training, but foster evidence may receive higher reliability for “in home” behavior. citeturn13view0  

**Inter-annotator agreement targets:**
- For categorical document-level labels: target κ ≥ 0.61 (“substantial”) for high-frequency labels; accept lower for rare/highly ambiguous traits during early phases. citeturn5search0turn5search4  
- For span-level tasks: measure agreement using span-oriented metrics (e.g., F1 overlap) rather than κ alone, since κ assumptions break for open-ended span annotation. citeturn5search18  

**Quality control workflow:**
- Dual annotation on an initial pilot set; adjudication by an expert reviewer; guideline revision; repeat until agreement stabilizes. This mirrors corpus-building best practices used in other high-stakes domains. citeturn5search18turn5search13  

**Annotation tools (suggestions):**
- entity["company","doccano","open-source annotation tool"] supports text classification and sequence labeling workflows suitable for this task. citeturn5search3turn5search15  
(Other comparable tools exist; select based on support for span annotation + adjudication + exports.)

**Estimated annotation volume (practical starting point):**
- Pilot: 300–500 listings (double-annotated) to refine ontology and guidelines.  
- Initial model: 3,000–8,000 listings with targeted oversampling of rare but critical behaviors (e.g., separation distress, cat incompatibility). This scale is typical when moving from heuristic extraction to robust supervised IE under class imbalance. citeturn2search0turn13view0turn18view0  

### Gold evaluation setup

**Splits:**
- Train/dev/test with **shelter-level split** when possible to measure generalization across writing styles. Shelters vary in formatting and language (paragraph vs bullet preferences and language nuances have measurable effects on reader interpretation). citeturn20view0  

**Primary metrics (extraction quality):**
- For each label and value: precision/recall/F1 at the **attribute assertion** level and at the **final normalized attribute** level.  
- For span detection: entity-level F1 with partial-overlap handling.

**Calibration metrics (confidence quality):**
- Expected Calibration Error (ECE) on held-out data; reliability diagrams; and per-label calibration. citeturn1search6turn1search2  
- Evaluate calibration both pre- and post-temperature scaling. citeturn1search2  

**Contradiction metrics:**
- Treat contradiction detection as a classification task: precision/recall/F1 for “true contradiction” vs “contextual difference” vs “temporal change.” Document-level NLI resources and fact verification datasets motivate and support this framing. citeturn4search1turn4search0  

**Thresholds for production (suggested operating points):**
- Use risk/coverage style reporting for “abstain to human review” decisions; selective prediction is a standard way to trade coverage for reduced error when confidence is low. citeturn21search7turn21search11  

### Human evaluation and error analysis

Because behavior descriptions are consequential and context-dependent, include structured human evaluation:

- **Adoption counselor review:** assess whether extracted attributes + evidence are sufficient to support counseling conversations, consistent with the recommendation to integrate multiple sources. citeturn13view0  
- **Error taxonomy:**  
  - Negation/scope errors (classic failure mode). citeturn1search0turn1search1  
  - Context confusion (kennel vs foster vs on-leash). citeturn23view0turn13view0  
  - Temporal confusion (“at first… now…”). citeturn16view1  
  - Marketing euphemism/omission bias (missing negatives, overstated positives). citeturn18view0turn9view0  

### Model comparison and ablation plan

Run controlled evaluations to justify complexity:

- Rules-only baseline (lexicons + negation rules). citeturn1search0turn6search2  
- CRF baseline vs transformer fine-tuning. citeturn6search0turn6search1  
- Add context tagging (NegEx/ConText style) → measure improvement in precision on negated/hedged statements. citeturn1search0turn1search1turn7search0  
- Add calibration (temperature scaling) → measure ECE change and downstream abstention reliability. citeturn1search2turn1search6  
- Add contradiction detection (rules then NLI) → measure contradiction precision and review load. citeturn1search3turn4search1  

This evaluation structure is designed to ensure the system remains **auditable and safe** in the face of known limitations of single assessments and the reality that behavior varies by context and time. citeturn13view0turn14view1turn16view1