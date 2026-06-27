---
doc_type: short
full_text: sources/nt_interpretation.md
---

# NT Interpretation Prompt Template

## Overview

This document contains a structured prompt template designed for use with a large language model acting as a **licensed neuropsychologist**. Its purpose is to auto-generate interpretive narrative paragraphs for individual domains within a neuropsychological evaluation report.

## Template Variables

| Variable | Description |
|---|---|
| `{domain}` | The cognitive or behavioral domain being interpreted (e.g., Memory, Attention, Executive Function) |
| `{score_lines}` | A summary of scores for the domain — subtests, scales, and associated metrics |

## Output Requirements

The prompt instructs the model to produce a **2–4 paragraph clinical narrative** with the following structure:

1. **Domain-level summary sentence** — States overall functioning level (e.g., "Cognitive abilities in the domain of X fell in the Y range.")
2. **Pattern description** — Discusses subtest/scale-level performance, noting intra-domain strengths and weaknesses
3. **Clinically significant scores** — Flags scores at or below the 5th percentile or at or above the 95th percentile
4. **Style constraints** — Third-person past tense, clinical but accessible language, flowing paragraphs (no bullet points)

## Clinical Context

This template is intended for integration into automated or semi-automated [[concepts/neuropsychological-assessment-automation]] workflows. It represents a domain-specific application of [[concepts/clinical-narrative-generation]] using structured prompting, and connects closely to the broader practice of [[concepts/neuropsychological-reporting]].

## Key Design Patterns

- **Role assignment**: The model is explicitly cast as a licensed neuropsychologist, anchoring tone and vocabulary consistent with [[concepts/clinical-communication-register]]
- **Structured input**: `{score_lines}` provides quantitative grounding for the narrative, drawing on [[concepts/neuropsychological-test-scores]]
- **Output constraints**: Paragraph format, tense, person, and clinical flagging thresholds are all specified
- **Percentile thresholds**: The 5th and 95th percentile cutoffs align with standard [[concepts/neuropsychological-score-interpretation]] conventions

## Potential Use Cases

- Automated report drafting as part of a [[concepts/neuropsychological-assessment-pipeline]]
- Training data generation for [[concepts/clinical-nlp-pipelines]]
- Standardization of report language across evaluators via [[concepts/neuropsychological-prompt-engineering]]

## Related Concepts
- [[concepts/narrative-report-generation]]
- [[concepts/cognitive-domains]]
- [[concepts/clinical-report-structure]]
- [[concepts/neuropsychological-prompt-configuration]]
- [[concepts/neuropsychological-report-variables]]
