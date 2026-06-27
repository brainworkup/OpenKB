---
sources: [summaries/redesign_20260623110910.md, summaries/sirf_synthesis.md, summaries/nt_interpretation.md, summaries/nse_narrative.md, summaries/neurocog.prompt.md, summaries/neurobehav.prompt.md]
brief: Structuring LLM prompts for formal neuropsychological report generation across all evaluation phases.
---

# Neuropsychological Prompt Engineering

Neuropsychological prompt engineering is the practice of structuring, parameterizing, and constraining large language model (LLM) prompts so that their output meets the standards of formal clinical neuropsychological reporting. It sits at the intersection of clinical expertise, natural language generation, and applied LLM configuration.

## Why Prompt Engineering Matters in Clinical Contexts

Neuropsychological reports are high-stakes documents. They inform diagnoses, treatment plans, school placements, disability determinations, and legal proceedings. Generated prose must be:

- **Clinically accurate** — faithfully representing test data and behavioral observations
- **Professionally formatted** — consistent with conventions of formal psychological reporting
- **Legally defensible** — precise enough to withstand scrutiny from other clinicians or courts
- **Accessible** — interpretable by families and non-specialist readers, not only other clinicians

Generic LLM prompts do not reliably satisfy these constraints. Domain-specific prompt engineering is required to close the gap.

## Prompt Family: Pre-Testing, Cognitive, Behavioral, Domain-Level Interpretation, and Synthesis

The neuropsychological prompt engineering ecosystem spans multiple complementary prompt documents, each targeting a distinct phase or domain of the evaluation workflow.

### `nse_narrative` — Neurobehavioral Status Exam Summary

The [[summaries/nse_narrative]] prompt targets the **pre-testing phase** of a neuropsychological evaluation. Rather than interpreting test scores, it synthesizes staged intake and historical information into a cohesive clinician-facing narrative — the Neurobehavioral Status Exam (NSE) summary. See [[concepts/neurobehavioral-status-exam]] for background on this clinical artifact.

The prompt is organized around four sequential evidence stages:

1. **Past records review** — prior records summary, medical history, psychiatric and psychological history, developmental and educational history, family history, and current medications
2. **Current intake and history materials** — intake and scales summary, subjective cognitive complaints, and functional impact areas to clarify through testing
3. **Tele-intake interview** — transcript or interview summary and behavioral observations recorded during the interview
4. **PHI-safe preprocessing** — redaction and preprocessing notes ensuring Protected Health Information compliance (see [[concepts/phi-data-handling]])

All stage inputs are injected via `{variable}` placeholders, making the prompt a reusable template that cleanly separates fixed clinical instructions from patient-specific data. The output is explicitly constrained to exclude diagnoses, differential diagnoses, treatment planning, and interpretation of objective test results — preserving the appropriate boundary between pre-testing narrative and later report sections. This reflects the [[concepts/staged-clinical-intake]] pattern of organizing evaluation information chronologically before testing begins.

### `neurocog` — Neurocognitive Performance

The [[summaries/neurocog.prompt]] document configures the model to interpret and summarize results from a ten-domain neuropsychological test battery:

1. General Cognitive Ability / Intelligence
2. Academic Skills
3. Verbal / Language Processing
4. Visual Perception / Construction
5. Memory
6. Attention / Executive Functioning
7. Motor / Sensorimotor
8. ADHD
9. Social Cognition
10. Emotional, Behavioral, and Personality Functioning

This prompt is oriented toward the interpretation of standardized cognitive test performance — the kind of data produced by instruments measuring IQ, working memory, processing speed, and visuospatial ability.

### `neurobehav` — Neurobehavioral and Socio-Emotional Functioning

The [[summaries/neurobehav.prompt]] document narrows focus to the behavioral, social, and emotional dimensions of the evaluation:

- ADHD and [[concepts/executive-function-deficits]]
- Social cognition and autism spectrum features (see [[concepts/autism-spectrum-disorder-clinical-features]])
- Emotional, behavioral, and personality functioning
- Adaptive functioning

See [[concepts/behavioral-rating-scales]] for the types of instruments whose data feeds these prompts.

### `nt_interpretation` — Domain-Level Score Interpretation

The [[summaries/nt_interpretation]] prompt addresses a more granular level of analysis: generating a **domain-specific interpretive narrative** from a structured set of subtest or scale scores. Rather than synthesizing an entire battery, this prompt takes a `{domain}` variable and a `{score_lines}` block as inputs and produces a focused 2–4 paragraph clinical interpretation for that single domain.

The output requirements are tightly specified:

1. **Opening domain summary** — A sentence stating overall functioning level (e.g., "Cognitive abilities in the domain of X fell in the Y range.")
2. **Score pattern description** — Subtest- and scale-level analysis noting intra-domain strengths and weaknesses
3. **Clinical significance flagging** — Explicit attention to scores at or below the 5th percentile or at or above the 95th percentile
4. **Style constraints** — Third-person past tense, clinical but accessible prose, no bullet points

This prompt complements the broader `neurocog` and `neurobehav` prompts by providing a reusable, domain-agnostic template that can be applied to any cognitive or behavioral domain where score data is available. It is a direct implementation of [[concepts/neuropsychological-score-interpretation]] conventions in prompt form.

### `sirf_synthesis` — Synthesis and Clinical Impressions

The [[summaries/sirf_synthesis]] prompt represents the **final integration phase** of the evaluation report pipeline. Where the preceding prompts generate domain-specific or phase-specific sections, the `sirf_synthesis` prompt is designed to unify all prior evidence into a single, coherent *Synthesis and Clinical Impressions* section — the capstone of a formal neuropsychological report.

The prompt accepts four injection variables:

- `{patient_name}` — Patient identifier
- `{nse_summary}` — The pre-testing narrative generated by the `nse_narrative` prompt
- `{nt_narrative}` — The neuropsychological test performance narrative generated by `neurocog` or `nt_interpretation`
- `{flagged_scores}` — Clinically significant scores at or below the 16th percentile or at or above the 84th percentile

The flagged score thresholds used here (≤16th or ≥84th percentile, approximately ±1 SD) are broader than the 5th/95th percentile cutoffs used in `nt_interpretation`, reflecting the synthesis section's role in capturing a wider range of clinically meaningful deviations for integrated interpretation. This design choice ensures that the synthesis catches meaningful patterns that domain-level narratives might flag only at a finer granularity.

The output is a **3–5 paragraph synthesis** that:

1. Integrates NSE clinical observations with objective test performance patterns
2. Identifies overarching cognitive strengths and areas of relative weakness across domains
3. Describes the functional and real-world impact of identified deficits
4. Places findings in the context of the referral question
5. Uses professional, DSM-5-TR aligned clinical language in flowing past-tense paragraphs

The `sirf_synthesis` prompt is explicitly scoped to cross-domain integration and referral question contextualization — tasks that no single domain prompt can accomplish alone. Its output draws on the outputs of all preceding prompt modules, making it the final node in the report generation pipeline. This reflects [[concepts/neuropsychological-synthesis]] as a distinct clinical skill encoded in prompt form.

Together, the `nse_narrative`, `neurocog`, `neurobehav`, `nt_interpretation`, and `sirf_synthesis` prompts form complementary modules within a complete evaluation report pipeline — covering pre-testing synthesis, full-battery cognitive performance, behavioral-emotional interpretation, granular domain-level score narratives, and final cross-domain synthesis respectively.

## Core Design Dimensions

### 1. Role Framing

Each prompt establishes a rich expert persona appropriate to its phase. The `nse_narrative` prompt instructs the model to write as a *licensed neuropsychologist* producing a pre-testing intake summary. The `neurocog` and `neurobehav` prompts establish a *board-certified clinical neuropsychologist* with subspecialty expertise across neurodevelopment, attention, ADHD, executive functioning, social-emotional development, ASD evaluation, and psychodiagnostic assessment. The `nt_interpretation` and `sirf_synthesis` prompts similarly cast the model as a *licensed neuropsychologist* — but each scoped to their respective task. This role framing shapes vocabulary selection, inferential depth, and interpretive tone throughout the output.

### 2. Domain Scoping and Phase Separation

Each prompt is scoped to a specific phase or domain to maintain focus and avoid hallucinated content from outside the relevant domain. The `nse_narrative` prompt is constrained to pre-testing synthesis and explicitly prohibits diagnostic interpretation. The `neurocog` prompt covers the full cognitive battery; the `neurobehav` prompt is limited to behavioral, social, and emotional domains. The `nt_interpretation` prompt is scoped to a single `{domain}` at a time. The `sirf_synthesis` prompt is explicitly scoped to cross-domain integration — it synthesizes across all prior sections rather than generating new domain-level data. This separation prevents scope creep and keeps each generated section coherent and focused.

### 3. Writing Style Constraints

Explicit stylistic instructions are embedded in all prompts to enforce report conventions:

| Constraint | Specification |
|---|---|
| Tense | Past tense throughout |
| Voice | Third-person |
| Register | Mixed technical/accessible, leaning professional |
| Format | Continuous paragraph prose (no bullet lists) |
| Pronouns | Patient-specific when available |

This reflects the broader concept of [[concepts/clinical-communication-register]] — calibrating language to serve both professional and lay audiences simultaneously, a principle also known as dual-audience design.

### 4. Percentile Thresholds and Clinical Significance

A consistent but deliberately varied feature across prompts is the use of **percentile cutoffs** as thresholds for clinical significance flags. The `nt_interpretation` prompt uses the **5th and 95th percentile** cutoffs for domain-level flagging. The `sirf_synthesis` prompt broadens this to the **16th and 84th percentile** (approximately ±1 SD) — appropriate for synthesis-level pattern recognition across the full battery. These cutoffs are standard conventions in [[concepts/neuropsychological-score-interpretation]] and their explicit encoding in the prompts prevents the model from applying different or inconsistent thresholds at each level of analysis.

### 5. Model Parameter Tuning

Clinical prompt engineering extends beyond text to LLM inference parameters. The `neurocog` and `neurobehav` prompts share a consistent parameter profile:

| Parameter | Value | Rationale |
|---|---|---|
| Temperature | 0.4 | Reduces variability; promotes precise clinical language |
| Max Tokens | 8192 | Accommodates complete, multi-paragraph domain summaries |
| Top-P | 1.0 | Full nucleus sampling, balanced by low temperature |
| Presence Penalty | 0.5 | Discourages repetitive topic introduction |
| Frequency Penalty | 0.5 | Reduces boilerplate phrase repetition |
| Mirostat | 0.8 | Dynamic perplexity control for coherent long-form output |
| Stop Sequence | `--END--` | Hard boundary preventing overgeneration |

These settings reflect the trade-off between creative flexibility and clinical reliability. See [[concepts/local-llm-inference]] for considerations when running such models on local hardware.

### 6. Template Variable Injection

Both the `{{{ input }}}` triple-brace pattern (unescaped Handlebars syntax) used in `neurocog` and `neurobehav`, and the `{variable}` single-brace pattern used in `nse_narrative`, `nt_interpretation`, and `sirf_synthesis`, serve the same architectural purpose: they are injection points for patient-specific or domain-specific data. The `sirf_synthesis` prompt uses `{patient_name}`, `{nse_summary}`, `{nt_narrative}`, and `{flagged_scores}` as its four primary injection variables, enabling the synthesis to be generated from the outputs of earlier pipeline stages rather than raw data alone. This pattern enables each prompt to function as a **reusable template** that separates fixed clinical instructions from variable patient data — a core principle of [[concepts/modular-report-architecture]].

## Prompt as a Modular Report Component

No individual prompt is a standalone artifact. Each represents one phase or domain module within a larger [[concepts/neuropsychological-assessment-pipeline]]. The `nse_narrative` prompt generates the pre-testing intake narrative; the `neurocog` prompt generates cognitive performance synthesis; the `neurobehav` prompt generates behavioral and social-emotional synthesis; the `nt_interpretation` prompt generates granular, domain-level score narratives; and the `sirf_synthesis` prompt integrates all prior sections into a final clinical impressions narrative. Together, these compose a complete [[concepts/clinical-report-structure]].

This modular design mirrors the architecture described in [[concepts/domain-processor-pattern]] and supports the [[concepts/narrative-report-generation]] workflow that transforms structured intake data and test scores into interpretive clinical prose.

## Relationship to Broader Concepts

- [[concepts/neuropsychological-reporting]] — The report-writing tradition this prompt engineering supports
- [[concepts/neuropsychological-report-variables]] — The patient variables (pronouns, scores, informant data) injected at runtime
- [[concepts/neuropsychological-assessment-automation]] — Automating report generation at scale using prompt pipelines
- [[concepts/neuropsychological-synthesis]] — The cross-domain clinical synthesis skill encoded in the `sirf_synthesis` prompt
- [[concepts/validity-language]] — Linguistic conventions for describing test validity and score interpretation
- [[concepts/yaml-configuration]] — The parameter block (temperature, penalties, stop tokens) uses YAML-style configuration
- [[concepts/role-based-llm-routing]] — Routing different report sections to appropriately specialized prompts
- [[concepts/neuropsychological-tests]] — The instruments whose scores feed into these prompt templates
- [[concepts/neuropsychological-test-scores]] — The structured score data passed via template injection
- [[concepts/pii-redaction-pipelines]] — Supporting PHI-safe preprocessing in the `nse_narrative` stage
- [[concepts/clinical-narrative-generation]] — The broader LLM application category this work exemplifies

## Key Principles Summary

1. **Phase specificity** — separate prompts govern pre-testing narrative (NSE), cognitive interpretation, behavioral interpretation, domain-level score narratives, and final cross-domain synthesis
2. **Expert persona grounding** produces more clinically appropriate vocabulary and inference
3. **Domain scoping** prevents hallucination and scope creep across report sections
4. **Explicit style constraints** enforce report conventions the model would not otherwise reliably follow
5. **Tiered percentile thresholds** are encoded directly in prompts — 5th/95th for domain-level flagging, 16th/84th for synthesis-level pattern recognition
6. **Parameter tuning** prioritizes precision over creativity for high-stakes clinical text
7. **Template injection** separates reusable clinical logic from patient-specific data
8. **Modular composition** allows individual phase and domain prompts to be combined into complete evaluation reports
9. **Pipeline chaining** in the `sirf_synthesis` prompt means later stages consume the outputs of earlier stages, not just raw data
10. **Consistent parameterization** across prompt modules ensures stylistic coherence across the full report

## See Also

- [[summaries/nse_narrative]] — Neurobehavioral Status Exam pre-testing narrative prompt
- [[summaries/neurocog.prompt]] — Neurocognitive performance interpretation prompt
- [[summaries/neurobehav.prompt]] — Neurobehavioral and socio-emotional interpretation prompt
- [[summaries/nt_interpretation]] — Domain-level score interpretation prompt template
- [[summaries/sirf_synthesis]] — Synthesis and Clinical Impressions integration prompt
- [[summaries/cognition.instructions]] — A related domain prompt for cognitive functioning
- [[summaries/neuropsych-narrative-writer]] — Narrative generation component in the broader pipeline
- [[summaries/clinical-validity-reviewer]] — QA layer applied to generated clinical prose


See also: [[summaries/redesign_20260623110910]]