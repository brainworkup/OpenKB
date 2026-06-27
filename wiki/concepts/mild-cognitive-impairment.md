---
sources: [summaries/bwu.neuro.reports.recs.books.aging.md, summaries/bwu.neuro.reports.recs.adhd.older-adult.md, summaries/report_body.md, summaries/NP-20240415-001_report.md]
brief: MCI is a clinical syndrome of cognitive decline beyond normal aging, not yet meeting dementia criteria.
---

# Mild Cognitive Impairment (MCI)

Mild Cognitive Impairment (MCI) is a clinical syndrome characterized by cognitive decline greater than expected for a person's age and education level, yet not significantly interfering with daily functional activities. MCI occupies an intermediate zone between normal aging and dementia, and is an important diagnostic target in [[concepts/neuropsychological-assessment-pipeline]] workflows.

## Subtypes

### Amnestic MCI
The most common subtype, defined by prominent memory impairment. Amnestic MCI carries elevated risk for progression to Alzheimer's disease. It is the subtype referenced in [[summaries/NP-20240415-001_report]] and [[summaries/report_body]], where below-average working memory performance on the WAIS-IV Digit Span subtest contributed to the clinical impression of early amnestic MCI.

### Non-Amnestic MCI
Involves impairment in cognitive domains other than memory (e.g., executive function, language, visuospatial ability), with relatively preserved episodic memory.

## Diagnostic Criteria

Core features of MCI typically include:

- **Subjective cognitive complaint** — reported by the patient, informant, or clinician
- **Objective cognitive impairment** — documented by standardized [[concepts/neuropsychological-tests]], typically ≥1–1.5 SD below normative means
- **Preserved functional independence** — activities of daily living remain largely intact
- **Not meeting criteria for dementia** — impairment does not reach the threshold for a major neurocognitive disorder

## Neuropsychological Detection

MCI is primarily identified through [[concepts/neuropsychological-reporting]] and formal cognitive testing. Relevant [[concepts/cognitive-domains]] that are assessed include:

- **Memory and learning** — often the earliest affected domain in amnestic MCI
- **Working memory** — assessed via tasks such as Digit Span; see [[concepts/working-memory]]
- **Attention and processing speed**
- **Executive function** — see [[concepts/executive-function-deficits]]
- **Language**
- **Visuospatial ability**

Score interpretation follows established classification frameworks documented in [[concepts/neuropsychological-test-scores]], with clinical concern typically flagged at or below the 16th percentile.

## Evidence from NP-20240415-001

In [[summaries/NP-20240415-001_report]] and the associated report body captured in [[summaries/report_body]], the evaluating system arrived at a preliminary clinical impression of **amnestic MCI** based on:

- WAIS-IV Digit Span scaled score of **9** (37th percentile) — classified as Average by norms but flagged as clinically significant given the referral context
- The Digit Span subtest's role as a [[concepts/working-memory-index]] contributor
- Limited evaluation scope (single subtest); broader confirmation would require a full battery

Notably, the [[summaries/NP-20240415-001_report]] report applies the same Digit Span result across six distinct cognitive domains (intelligence, memory, attention, executive function, language, and visuospatial ability), illustrating a key limitation of single-subtest evaluations: cross-domain inferences drawn from one working memory measure are speculative and should be clearly qualified using appropriate [[concepts/validity-language]].

An additional inconsistency observed in the report is a mismatch between the normative classification label ("Average") and the clinical significance statement ("Below average, ≤16th percentile"). The 37th percentile falls squarely within the Average range by standard neuropsychological conventions; it does not fall at or below the 16th percentile. This type of internal inconsistency highlights the importance of clear, consistent score interpretation in [[concepts/clinical-report-structure]] and careful language choices in [[concepts/narrative-report-generation]] pipelines. Automated systems must be engineered to maintain coherence between numerical values and their descriptive labels.

The report also lacked key contextual data — including referral reason, patient age, education level, medical history, and presenting concerns — further limiting the validity of the diagnostic impression. See [[concepts/neuropsychological-report-variables]] for the full set of variables that should populate a complete neuropsychological report.

## Recommended Follow-Up

Standard clinical recommendations for suspected MCI, as outlined in the NP-20240415-001 report, include:

1. **Neuroimaging** (MRI) — to rule out structural causes
2. **Reassessment in 12 months** — to monitor for progression toward dementia
3. **Cognitive rehabilitation referral** — to support maintained function

## Limitations in Single-Subtest Evaluations

A diagnosis of MCI requires convergent evidence across multiple cognitive domains. Relying on a single subtest (as in the NP-20240415-001 evaluation) is insufficient for a definitive diagnosis. A comprehensive [[concepts/neuropsychological-assessment-pipeline]] incorporating a full test battery is required for valid clinical conclusions. Language used in reports must reflect appropriate uncertainty; see [[concepts/validity-language]].

The practice of mapping one subtest result across all cognitive domains — as seen in [[summaries/NP-20240415-001_report]] — should be treated as a structural placeholder rather than a clinically meaningful multi-domain assessment. Automated [[concepts/narrative-report-generation]] systems must guard against over-extrapolation from limited data. This concern is directly addressed in the design principles documented in [[concepts/neuropsychological-assessment-automation]] and [[concepts/neuropsychological-synthesis]].

## Related Concepts

- [[concepts/neuropsychological-reporting]] — Report structure and narrative conventions
- [[concepts/neuropsychological-tests]] — Standardized instruments used in MCI detection
- [[concepts/neuropsychological-test-scores]] — Score interpretation frameworks
- [[concepts/cognitive-domains]] — The functional areas assessed in neuropsychological evaluation
- [[concepts/working-memory]] — Key domain implicated in amnestic MCI
- [[concepts/working-memory-index]] — Composite index from the WAIS-IV relevant to MCI detection
- [[concepts/wais-iv]] — The primary instrument referenced in NP-20240415-001
- [[concepts/executive-function-deficits]] — Non-amnestic MCI may present with executive dysfunction
- [[concepts/clinical-report-structure]] — How findings are organized in formal reports
- [[concepts/narrative-report-generation]] — AI-assisted generation of neuropsychological narratives
- [[concepts/validity-language]] — Appropriate hedging and diagnostic language in reports
- [[concepts/neuropsychological-assessment-pipeline]] — End-to-end workflow for cognitive evaluation
- [[concepts/neuropsychological-report-variables]] — Variables required for a complete report
- [[concepts/neuropsychological-synthesis]] — Cross-domain integration of test findings
- [[concepts/neuropsychological-assessment-automation]] — Automated pipeline design considerations


See also: [[summaries/bwu.neuro.reports.recs.adhd.older-adult]]

See also: [[summaries/bwu.neuro.reports.recs.books.aging]]