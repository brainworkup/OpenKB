---
doc_type: short
full_text: sources/neurocog.prompt.md
---

# neurocog.prompt — Neurocognitive Performance Interpretation Prompt

## Overview

This is a structured system prompt designed for use by (or in emulation of) a **board-certified clinical neuropsychologist**. It configures a language model to interpret and summarize neurocognitive test battery results in a professional, integrated narrative format. The prompt targets clinical reporting workflows where raw test data must be translated into readable, domain-organized summaries.

## Model Configuration

| Parameter | Value |
|---|---|
| Temperature | 0.4 |
| Max Tokens | 8192 |
| Top-P | 1.0 |
| Presence Penalty | 0.5 |
| Frequency Penalty | 0.5 |
| Mirostat | 0.8 |
| Stop Sequence | `--END--` |

The relatively low temperature (0.4) encourages consistent, precise clinical language. Presence and frequency penalties reduce repetition, important for multi-domain reports. The high token ceiling (8192) accommodates comprehensive neuropsychological write-ups.

## Clinical Persona & Expertise

The system prompt establishes the model as an expert in:

- [[concepts/neuropsychological-assessment-automation]] — Psychodiagnostic and neuropsychological examination
- [[concepts/executive-function-deficits]] — Attention, ADHD, and executive functioning
- [[concepts/autism-spectrum-disorder-clinical-features]] — ASD and Asperger's syndrome evaluation
- Social-emotional development and social cognition

## Neurocognitive Domains Covered

The prompt references a standardized test battery spanning ten domains:

1. **General Cognitive Ability / Intelligence**
2. **Academic Skills**
3. **Verbal / Language Processing**
4. **Visual Perception / Construction**
5. **Memory**
6. **Attention / Executive Functioning**
7. **Motor / Sensorimotor**
8. **ADHD**
9. **Social Cognition**
10. **Emotional, Behavioral, and Personality Functioning**

This maps closely to standard [[concepts/cognitive-domains]] frameworks used in clinical and forensic neuropsychological evaluation settings. See also [[concepts/neuropsychological-tests]] and [[concepts/neuropsychological-test-scores]] for related domain coverage.

## Output Instructions

The prompt directs the model to produce summaries with the following characteristics:

- **Third-person, past-tense voice** — consistent with formal [[concepts/clinical-report-structure]] conventions
- **Mixed technical and lay language** — accessible to both professionals and non-professionals, following [[concepts/dual-audience-design]] principles, with a professional lean per [[concepts/clinical-communication-register]]
- **Complete and sophisticated** — no superficial or incomplete coverage
- **Paragraph form** — integrated [[concepts/narrative-report-generation]] rather than bullet lists
- **Pronoun-sensitive** — uses the patient's stated pronouns when available

## Use Case

This prompt is intended to be used with a dynamic input (`{{{ input }}}`) containing raw or structured neurocognitive test results for a specific patient. The model then synthesizes these results into a domain-specific integrated summary, suitable for inclusion in a formal neuropsychological report. The configuration is part of the broader [[concepts/neuropsychological-prompt-configuration]] and [[concepts/neuropsychological-prompt-engineering]] framework, and connects directly to [[concepts/neuropsychological-reporting]] and [[concepts/modular-report-architecture]] workflows.

## Key Concepts

- [[concepts/neuropsychological-prompt-configuration]]
- [[concepts/neuropsychological-prompt-engineering]]
- [[concepts/cognitive-domains]]
- [[concepts/neuropsychological-reporting]]
- [[concepts/narrative-report-generation]]
- [[concepts/clinical-report-structure]]
- [[concepts/dual-audience-design]]
- [[concepts/clinical-communication-register]]
- [[concepts/autism-spectrum-disorder-clinical-features]]
- [[concepts/executive-function-deficits]]
- [[concepts/yaml-configuration]]

## Related Concepts
- [[concepts/behavioral-rating-scales]]
- [[concepts/local-llm-inference]]
- [[concepts/validity-language]]
