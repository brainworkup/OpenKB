---
doc_type: short
full_text: sources/neurobehav.prompt.md
---

# neurobehav — Behavioral & Personality Assessment Prompt

## Overview

This document defines a structured prompt template (`neurobehav`) used by a clinical neuropsychologist AI assistant to generate integrated written summaries of patient behavioral rating scale data. It is designed for use in psychodiagnostic and neuropsychological evaluation reports.

## Purpose & Role

The prompt configures the AI to act as an **experienced, board-certified clinical neuropsychologist** with expertise in:

- [[concepts/executive-function-deficits]] — ADHD, attention, and executive functioning assessment
- [[concepts/autism-spectrum-disorder-clinical-features]] — ASD and Asperger's syndrome evaluation
- Neurodevelopmental assessment across the lifespan
- [[concepts/neuropsychological-assessment-pipeline]] — Psychiatric, personality, and psychodiagnostic evaluation
- Social-emotional development

## Assessment Domains Covered

The prompt is specifically scoped to [[concepts/behavioral-rating-scales]] across four domains:

1. **ADHD / Executive Functioning** — Attention regulation, inhibition, [[concepts/working-memory]], cognitive flexibility
2. **Social Cognition** — Theory of mind, social awareness, social communication
3. **Emotional, Behavioral, and Personality Functioning** — Mood, affect, behavioral patterns, personality traits
4. **Adaptive Functioning** — Daily living skills, independence, functional competence

Rating scales may be completed by multiple informants: the patient (self-report), parents, teachers, and clinicians.

## Writing Style Instructions

The generated output must adhere to specific stylistic requirements:

- **Tense**: Past tense throughout
- **Voice**: Third-person perspective
- **Register**: Mixed technical and accessible language, leaning professional — interpretable by both clinicians and non-specialists; see [[concepts/dual-audience-design]] and [[concepts/clinical-communication-register]]
- **Format**: Continuous paragraph-form prose (not bullet lists)
- **Completeness**: Sophisticated, integrated, and clinically complete
- **Pronouns**: Patient-specific pronouns used when available

## Model Parameters

| Parameter | Value |
|---|---|
| Temperature | 0.4 |
| Max Tokens | 8192 |
| Top-P | 1.0 |
| Presence Penalty | 0.5 |
| Frequency Penalty | 0.5 |
| Mirostat | 0.8 |
| Stop Sequence | `--END--` |

The relatively low temperature (0.4) and moderate penalty settings reflect a preference for **consistent, clinically precise language** over creative variation — appropriate for formal report writing. These settings align with [[concepts/neuropsychological-prompt-engineering]] principles for reproducible clinical outputs.

## Input/Output Structure

- **Input**: Raw or summarized behavioral rating scale data injected via `{{{ input }}}` (triple-brace Handlebars-style template variable, unescaped)
- **Output**: An integrated narrative summary paragraph suitable for inclusion in a formal neuropsychological evaluation report

This input/output structure is consistent with the broader [[concepts/narrative-report-generation]] and [[concepts/modular-report-architecture]] patterns used across the neuropsychological reporting system.

## Clinical Context

This prompt is part of a broader neuropsychological report writing workflow — see [[concepts/neuropsychological-reporting]] — likely paired with other domain-specific prompts (e.g., cognitive, memory, language) that together compose a full evaluation report. The behavioral and personality domain represented here addresses the **affective and social** dimensions of neuropsychological functioning. It connects closely to [[concepts/clinical-report-structure]] and [[concepts/neuropsychological-assessment-automation]].

## Key Design Principles

- Emphasizes **integration** across multiple informant perspectives
- Balances clinical rigor with accessibility for families and non-specialist readers, per [[concepts/dual-audience-design]]
- Enforces formal report conventions (tense, voice, format)
- Scoped narrowly to behavioral/personality/adaptive domains to maintain focus
- Relies on [[concepts/yaml-configuration]] for model parameter control

## Related Concepts
- [[concepts/cognitive-domains]]
- [[concepts/style-profiles]]
- [[concepts/pai-assessment]]
