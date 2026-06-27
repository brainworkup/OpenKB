---
sources: [summaries/clinical-assessment.md, summaries/attention-problems.md, summaries/report_body.md, summaries/NP-20240415-001_report.md, summaries/neuropsych-pdf-parser.md, summaries/AGENTS_luria.md]
brief: Standardized tools measuring cognitive and psychological functioning, forming the empirical core of neuropsychological evaluations.
---

# Neuropsychological Tests & Instruments

Neuropsychological tests are standardized tools used by clinicians and researchers to measure cognitive, behavioral, and psychological functioning. They form the empirical backbone of neuropsychological evaluations, producing quantitative scores that guide diagnosis, treatment planning, and progress monitoring.

## Role in Document Processing

In automated pipelines such as the one described in [[summaries/AGENTS_luria]], the ability to **recognize and classify** neuropsychological instruments is a core function of the ingestion stage. The PDF Parser Worker uses instrument names as classification signals to identify documents as Neuropsychological Assessment Reports — distinguishing them from clinical notes, raw data sheets, or research papers.

As illustrated by [[summaries/NP-20240415-001_report]], a real evaluation may include as few as a single subtest (e.g., WAIS-IV Digit Span), making instrument recognition and subtest-level granularity equally important for downstream processing.

## Common Instruments

The following tests are explicitly referenced as classification anchors in the ingestion pipeline:

### Cognitive & Intelligence
- **WAIS-IV** (Wechsler Adult Intelligence Scale, 4th Ed.) — measures general intelligence across verbal, perceptual, working memory, and processing speed domains; individual subtests such as Digit Span contribute to composite indices like the Working Memory Index
- **MMSE** (Mini-Mental State Examination) — brief cognitive screening
- **MoCA** (Montreal Cognitive Assessment) — sensitive screening for [[concepts/mild-cognitive-impairment]]
- **RBANS** (Repeatable Battery for the Assessment of Neuropsychological Status) — broad cognitive battery

### Memory
- **WMS** (Wechsler Memory Scale) — assesses multiple memory domains including immediate, delayed, and working memory

### Executive Function
- **WCST** (Wisconsin Card Sorting Test) — measures cognitive flexibility and set-shifting
- **BRIEF** (Behavior Rating Inventory of Executive Function) — behavioral rating of executive functioning in daily life
- **Trail Making Test** (Parts A & B) — assesses processing speed, attention, and cognitive flexibility

### Attention & Processing Speed
- **CPT** (Continuous Performance Test) — measures sustained attention and impulsivity
- **Stroop Test** — assesses selective attention and cognitive control (interference)

### Mood & Affect
- **BDI** (Beck Depression Inventory) — self-report measure of depressive symptom severity

## Score Types

Neuropsychological tests produce multiple score formats, all of which must be preserved exactly during document ingestion (see [[summaries/AGENTS_luria]]). As demonstrated in [[summaries/NP-20240415-001_report]], a single subtest may yield several score types simultaneously:

| Score Type | Description | Example (Digit Span) |
|---|---|---|
| **Raw Score** | Unscaled count of correct responses or items | 16 |
| **Scaled Score** | Age-normed score, typically M=10, SD=3 | 9 |
| **Standard Score** | Normalized score, typically M=100, SD=15 | N/A for subtests |
| **T-Score** | Normalized score, M=50, SD=10 | N/A for subtests |
| **Percentile Rank** | Position relative to normative sample | 37th |
| **Standard Deviation** | Deviation from normative mean | — |

These values feed directly into [[concepts/neuropsychological-test-scores]] and downstream [[concepts/neuropsychological-reporting]] processes.

## Composite Indices

Individual subtests often aggregate into **composite index scores** that represent broader cognitive constructs. The WAIS-IV Digit Span, for example, contributes to the **Working Memory Index** — a composite reflecting the capacity to hold and manipulate information in short-term memory. The relationship between subtest scaled scores and composite indices is critical for [[concepts/neuropsychological-report-variables]] and for [[concepts/narrative-report-generation]] agents that must interpret scores in clinical context.

Understanding composite structures also supports [[concepts/validity-language]] decisions: when only one subtest from an index is administered, the composite index cannot be formally calculated, and the report must flag this as a limitation.

## Cognitive Domains Assessed

Neuropsychological instruments map onto broad [[concepts/cognitive-domains]] including:

- **Intelligence / General Ability**
- **Attention & Concentration**
- **Executive Function**
- **Memory (Verbal & Visual)** — including [[concepts/working-memory]]
- **Language**
- **Visuospatial Processing**
- **Processing Speed**
- **Mood & Emotional Functioning**

In practice, report templates often organize findings by these domains even when the evaluation is limited. As shown in [[summaries/NP-20240415-001_report]], a single test result may be reported under multiple domain headings, with appropriate caveats regarding the scope of assessment.

## Scope Limitations & Evaluation Completeness

A recurring challenge in both clinical practice and automated report generation is evaluations of **limited scope** — where only a subset of a test battery is administered. This creates several downstream challenges:

- Composite indices may be incomplete or formally incalculable
- Domain-level conclusions must be qualified
- Recommendations for reassessment or expanded testing are typically warranted
- Automated narrative generators (see [[concepts/narrative-report-generation]]) must detect and articulate these limitations using appropriate [[concepts/validity-language]]

The [[concepts/clinical-report-structure]] for such evaluations should include an explicit **Limitations & Caveats** section, as illustrated by the NP-20240415-001 report.

## Pipeline Integration

In automated neuropsychological workflows, instrument recognition is the first classification step. The structured output from the ingestion worker (see [[summaries/AGENTS_luria]]) feeds into the broader [[concepts/neuropsychological-assessment-pipeline]], where scores are extracted, normalized, and incorporated into clinical reports via [[concepts/neuropsychological-reporting]] agents.

Accurate identification of test instruments — including subtest names and their parent batteries — also enables downstream natural language processing tools (see [[concepts/clinical-nlp-pipelines]]) to locate, extract, and validate score tables within documents. The [[concepts/pdf-score-extraction]] stage depends on correct instrument name matching to anchor score lookups within document text.

## Related Pages

- [[summaries/AGENTS_luria]] — the ingestion worker that classifies documents by instrument type
- [[summaries/NP-20240415-001_report]] — example evaluation using a single WAIS-IV subtest
- [[concepts/neuropsychological-assessment-pipeline]] — the multi-stage pipeline these instruments feed into
- [[concepts/neuropsychological-test-scores]] — detailed handling of score types and formats
- [[concepts/neuropsychological-reporting]] — downstream reporting from extracted test data
- [[concepts/neuropsychological-report-variables]] — variables derived from test scores for report generation
- [[concepts/cognitive-domains]] — the functional domains these tests measure
- [[concepts/clinical-nlp-pipelines]] — NLP tools used to parse instrument data from text
- [[concepts/phi-data-handling]] — privacy handling for patient scores and identifiers
- [[concepts/narrative-report-generation]] — automated generation of clinical narratives from scores
- [[concepts/validity-language]] — language patterns used to qualify findings and flag limitations
- [[concepts/mild-cognitive-impairment]] — a key diagnostic category assessed via these instruments
- [[concepts/working-memory]] — a domain prominently measured by WAIS-IV Digit Span
- [[concepts/clinical-report-structure]] — organizational conventions for test result reporting
- [[summaries/neuropsych-pdf-parser]] — PDF parsing stage that identifies instrument tables

See also: [[summaries/report_body]]

See also: [[summaries/attention-problems]]

See also: [[summaries/clinical-assessment]]