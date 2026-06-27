---
sources: [summaries/Apply-to-Y-Combinator-JWT.md, summaries/2026-06-26-2133-plan.md, summaries/redesign_20260623110910.md, summaries/redesign_20260623110817.md, summaries/bwu.neuro.reports.recs.behav-mod.md, summaries/bwu.neuro.reports.recs.adhd.dref-intervention-strategies.md, summaries/DIAGNOSIS_PARSER_IMPROVEMENTS.md, summaries/AUTISM_RAG_SYSTEM_DOCUMENTATION.md, summaries/CARS2-Manual_extracted.md, summaries/pai_14.md, summaries/pai_07.md, summaries/pai_02.md, summaries/pai_01.md, summaries/pai_00.md, summaries/sirf_synthesis.md, summaries/nt_interpretation.md, summaries/nse_narrative.md, summaries/neurocog.prompt.md, summaries/neurobehav.prompt.md, summaries/0007-voice-modular-report-sections-via-quarto-includes.md, summaries/customization.md, summaries/style-training-to-report-drafting.md, summaries/report-rendering-pipeline.md, summaries/style-trainer.md, summaries/style-extensions.md, summaries/soul-style-agent.md, summaries/report-template.md, summaries/report-generator.md, summaries/0010-voice-quarto-typst-reporting.md, summaries/0007-style-modular-report-sections-via-quarto-includes.md, summaries/0005-style-quarto-custom-format-extensions-for-report-variants.md, summaries/multi_patient_transcript.md, summaries/report_body.md, summaries/NP-20240415-001_report.md, summaries/README.md, summaries/neuropsych-narrative-writer.md, summaries/clinical-validity-reviewer.md, summaries/issue_branding_typst.md, summaries/README_PIPELINE.md, summaries/README_AS_PROCESSING.md, summaries/AS_PROCESSING_COMPLETE.md, summaries/brainworkup-brand-voice-guide.md, summaries/template-system.md, summaries/quarto-extensions.md, summaries/overview.md, summaries/003-modular-template-structure.md, summaries/001-choose-quarto-typst.md, summaries/SKILL.md]
brief: Standardized organizational framework for presenting neuropsychological evaluation findings in professional reports.
---

# Clinical Report Structure

Clinical report structure refers to the standardized organizational framework used to present neuropsychological evaluation findings in a coherent, professionally acceptable format. A consistent structure ensures that referral sources, clinicians, educators, and other stakeholders can locate key information efficiently and that reports meet professional and medicolegal standards.

## Pre-Testing Narrative: The Neurobehavioral Status Exam (NSE)

Before formal neuropsychological testing begins, the evaluation report opens with a **Neurobehavioral Status Exam (NSE) summary** — a clinician-facing pre-testing narrative that synthesizes all available intake and historical information. This section is distinct from the test results section; it does not interpret objective scores or render diagnoses. Instead, it contextualizes the referral, documents relevant history, describes behavioral observations, and clarifies what domains subsequent testing should explore.

The [[concepts/neurobehavioral-status-exam]] section is built from a staged evidence architecture, as illustrated in the `nse_narrative` prompt template (see [[summaries/nse_narrative]]):

- **Stage 1 — Past Records Review**: Prior records summary, medical history, psychiatric and psychological history, developmental and educational history, family history, and current medications.
- **Stage 2 — Current Intake Materials**: Intake and history summaries, subjective cognitive complaints, and functional impact areas to clarify through testing.
- **Stage 3 — Tele-Intake Interview**: Transcript or interview summary and behavioral observations recorded during the session.
- **Stage 4 — PHI-Safe Preprocessing**: Redaction and preprocessing notes ensuring Protected Health Information compliance before the narrative is drafted.

The NSE summary must be written in professional clinical prose using past tense, organized into flowing paragraphs without bullet points, and must strictly exclude diagnoses, differential diagnoses, recommendations, treatment planning, or any interpretation of objective test results. This scope limitation preserves the appropriate boundary between the pre-testing narrative and later report sections.

This [[concepts/staged-clinical-intake]] model aligns with the [[concepts/neuropsychological-prompt-engineering]] approach used throughout the Luria pipeline, where structured prompt templates with injected variables drive AI-assisted narrative generation. The NSE prompt template uses `{variable}` placeholders for all patient-specific fields, making it reusable across cases while supporting [[concepts/phi-data-handling]] compliance via its dedicated preprocessing stage.

## Canonical Six-Section Format

As defined in the Luria report-writing skill (see [[summaries/SKILL]]), the standard report follows six ordered sections:

### 1. Reason for Referral
Establishes clinical context: who referred the patient, what questions prompted the evaluation, and what diagnostic or functional concerns are being addressed.

### 2. Sources Reviewed
Documents all informational inputs consulted — prior records, school reports, medical history, collateral interviews, and any other sources that informed the evaluation.

### 3. Behavioral Observations
Qualitative narrative describing the evaluee's presentation during testing: affect, cooperation, effort, fatigue, and any factors that may have affected performance validity.

### 4. Test Results by Domain
Organizes quantitative findings by [[concepts/cognitive-domains]] (e.g., memory, attention, language, executive function). Raw and standard scores are carried by tables; prose interprets patterns rather than restating numbers.

### 5. Summary and Impressions
Integrated clinical interpretation synthesizing all data sources into a coherent diagnostic picture. This section is the conceptual core of the report and is described in detail below.

### 6. Recommendations
Actionable guidance for intervention, accommodation, further evaluation, or follow-up — directly tied to the findings and impressions.

## Synthesis and Clinical Impressions Section

The **Synthesis and Clinical Impressions** section — sometimes called the SIRF synthesis — is the integrative culmination of the evaluation report. It is produced after all test data, NSE observations, and flagged scores have been gathered, and it serves a distinct function from the domain-by-domain test results section.

The `sirf_synthesis` prompt template (see [[summaries/sirf_synthesis]]) operationalizes this section via a structured LLM prompt that accepts four injectable variables:
- `{patient_name}` — patient identifier
- `{nse_summary}` — NSE clinical observations narrative
- `{nt_narrative}` — neuropsychological test performance narrative
- `{flagged_scores}` — clinically significant scores at or below the 16th percentile or at/above the 84th percentile

The resulting 3–5 paragraph synthesis is expected to:
1. **Integrate** NSE clinical observations with objective test performance patterns
2. **Identify** overarching cognitive strengths and areas of relative weakness across domains
3. **Describe** the functional and real-world impact of identified deficits
4. **Place** findings in the context of the referral question
5. **Use** professional, DSM-5-TR aligned clinical language in flowing past-tense paragraphs

The flagged score thresholds (≤16th or ≥84th percentile) correspond to approximately ±1 SD from the mean — the standard neuropsychological cutoff for clinically meaningful deviation. This integration of qualitative (NSE observations) and quantitative (test scores) data is central to [[concepts/neuropsychological-synthesis]] and the [[concepts/neuropsychological-assessment-pipeline]].

Because the synthesis section must contextualize findings relative to the original referral question, it functions as the bridge between domain-specific test results and the Recommendations section that follows. It is also the section most dependent on [[concepts/neuropsychological-prompt-engineering]] — the LLM is explicitly cast in the role of a licensed neuropsychologist to shape tone, vocabulary, and interpretive framing consistent with [[concepts/clinical-narrative-generation]] conventions.

## Forensic Neuropsychological Report Extensions

When a report serves medicolegal purposes, the six-section core is extended with additional sections required by forensic practice. The `typst-report-formatter` skill (see [[summaries/SKILL]]) defines these additions as part of the [[concepts/forensic-neuropsychological-evaluation]] report format:

- **Limitations and Caveats** — Documents effort and validity concerns, incomplete battery issues, and cultural or linguistic factors that qualify the findings.
- **Forensic Opinion** — A formally presented opinion statement rendered within a reasonable degree of neuropsychological certainty, visually set apart (e.g., in a shaded box) to distinguish it from clinical impressions.

The full forensic section sequence is:
1. Introduction and Purpose
2. Records Reviewed
3. Background Information
4. Tests Administered
5. Neuropsychological Findings (with domain subsections)
6. Cognitive Profile Summary
7. Clinical Impressions and Diagnostic Formulation
8. Recommendations
9. Limitations and Caveats
10. Forensic Opinion

Diagnostic formulation in forensic reports aligns with DSM-5/ICD-11 classification, and differential diagnoses are included when clinically relevant.

## AI-Generated Report Structure: Observed Patterns

The document [[summaries/NP-20240415-001_report]] and [[summaries/report_body]] together illustrate how AI-generated neuropsychological narrative reports implement the canonical structure when derived from structured JSON input. Several structural observations are notable:

### Domain-by-Domain Organization
The AI report expands the "Test Results by Domain" section into six explicit subsections — Intelligence/General Cognitive Ability, Memory & Learning, Attention & Processing Speed, Executive Function, Language, and Visuospatial Ability — each carrying standardized field-level metadata:

- Test name and subtest
- Raw score, Scaled Score, Standard Score, T-Score
- Percentile Rank
- Classification label
- Composite Index membership
- Prose interpretation
- Clinical Significance flag
- Discrepancy notation

This fine-grained per-domain structure aligns with [[concepts/neuropsychological-test-scores]] conventions and the [[concepts/modular-report-architecture]] approach.

### Handling Sparse Data
When evaluation data are limited to a single subtest — as in NP-20240415-001, where only the [[concepts/wais-iv]] Digit Span was administered — the AI-generated report populates all domain sections with the available data rather than omitting sections. This ensures structural completeness while the Limitations & Caveats section explicitly flags the narrow scope of the battery.

This pattern reflects a key design tension in [[concepts/narrative-report-generation]]: preserving report shape integrity when underlying data are incomplete. The [[summaries/report_body]] document demonstrates this concretely — a single [[concepts/working-memory-index]] subtest result is propagated across all six cognitive domain subsections, with identical interpretation in each. This approach maintains structural completeness but requires explicit acknowledgment in the Limitations section and highlights the need for clinician review before any report is finalized.

### Classification Label Inconsistency
A notable artifact observed in both [[summaries/NP-20240415-001_report]] and [[summaries/report_body]] is the tension between the formal classification label ("Average," corresponding to a scaled score of 9 at the 37th percentile) and the clinical significance statement ("below average, ≤16th percentile"). In the NP-20240415-001 case, the WAIS-IV Digit Span scaled score of 9 falls squarely within the Average range and does not meet the standard ≤16th percentile threshold for clinical significance. This inconsistency between the classification label and the narrative interpretation is a meaningful data quality signal.

This pattern underscores a critical requirement in [[concepts/neuropsychological-reporting]]: AI-generated drafts must undergo clinician review to catch mismatches between score labels, narrative characterizations, and clinical significance thresholds. The [[concepts/report-review-qa]] process should specifically include a check that percentile ranks, classification labels, and clinical significance flags are mutually consistent before any report is finalized or transmitted.

### Incomplete Background Context
The NP-20240415-001 report illustrates a second structural gap: when referral reason, background information, and patient demographics are absent from the source data, the AI system correctly marks those sections as "N/A" rather than fabricating content. This "graceful degradation" behavior — maintaining section headers while acknowledging data absence — is preferable to omission or confabulation, but it highlights the importance of complete [[concepts/staged-clinical-intake]] data collection prior to report generation.

### Recommendations Section as Standalone Output
Even when test coverage is severely limited, the Recommendations section remains substantive in AI-generated reports. In NP-20240415-001, three concrete recommendations were generated (neuroimaging, reassessment at 12 months, cognitive rehabilitation referral) despite the minimal assessment battery. This reflects the value of embedding clinical decision logic into the [[concepts/neuropsychological-prompt-configuration]] layer, ensuring that standard-of-care recommendations are surfaced even when data inputs are sparse.

## Related Concepts

- [[concepts/neuropsychological-reporting]]
- [[concepts/modular-report-architecture]]
- [[concepts/narrative-report-generation]]
- [[concepts/neuropsychological-synthesis]]
- [[concepts/clinical-narrative-generation]]
- [[concepts/neuropsychological-assessment-pipeline]]
- [[concepts/cognitive-domains]]
- [[concepts/neuropsychological-test-scores]]
- [[concepts/forensic-neuropsychological-evaluation]]
- [[concepts/staged-clinical-intake]]
- [[concepts/phi-data-handling]]
- [[concepts/report-review-qa]]
- [[concepts/wais-iv]]
- [[concepts/working-memory-index]]
- [[concepts/mild-cognitive-impairment]]

See also: [[summaries/README]]

See also: [[summaries/pai_00]]

See also: [[summaries/pai_01]]

See also: [[summaries/pai_02]]

See also: [[summaries/pai_07]]

See also: [[summaries/pai_14]]

See also: [[summaries/CARS2-Manual_extracted]]

See also: [[summaries/AUTISM_RAG_SYSTEM_DOCUMENTATION]]

See also: [[summaries/DIAGNOSIS_PARSER_IMPROVEMENTS]]

See also: [[summaries/bwu.neuro.reports.recs.adhd.dref-intervention-strategies]]

See also: [[summaries/bwu.neuro.reports.recs.behav-mod]]

See also: [[summaries/redesign_20260623110817]]

See also: [[summaries/redesign_20260623110910]]

See also: [[summaries/2026-06-26-2133-plan]]

See also: [[summaries/Apply-to-Y-Combinator-JWT]]