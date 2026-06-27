---
doc_type: short
full_text: sources/nse_narrative.md
---

# NSE Narrative Prompt Template

## Overview

This document contains a structured prompt template designed for use with a large language model (LLM) acting as a licensed neuropsychologist. Its purpose is to generate a **Neurobehavioral Status Exam (NSE) summary** — the pre-testing narrative section of a comprehensive [[concepts/neuropsychological-reporting]] report.

## Purpose and Clinical Context

The NSE summary synthesizes intake and historical information gathered before formal neuropsychological testing begins. It serves as a clinician-facing document that contextualizes why a patient was referred, what their history reveals, and what domains the subsequent testing should explore. It explicitly excludes diagnoses, differential diagnoses, treatment planning, and interpretation of objective test results. This output is a core component of the [[concepts/neurobehavioral-status-exam]] workflow and feeds into the broader [[concepts/neuropsychological-assessment-pipeline]].

## Staged Evidence Architecture

The prompt is organized into four sequential input stages, each populated via template variables, reflecting the [[concepts/staged-clinical-intake]] design pattern:

### Stage 1 — Past Records Review
- Prior records summary
- Medical history
- Psychiatric and psychological history
- Developmental and educational history
- Family history
- Current medications

### Stage 2 — Current Intake and History Materials
- Intake, history, and scales summary
- Subjective cognitive complaints reported by the patient
- Functional impact areas to clarify through testing

### Stage 3 — Tele-Intake Interview
- Transcript or interview summary
- Behavioral observations recorded during the interview

### Stage 4 — PHI-Safe Preprocessing
- Redaction or preprocessing notes to ensure Protected Health Information (PHI) compliance, consistent with [[concepts/phi-data-handling]] and [[concepts/pii-redaction-pipelines]]

## Output Requirements

The generated NSE summary must:
- Synthesize all staged evidence into a cohesive pre-testing narrative
- Describe referral context, salient history, behavioral observations, and current concerns
- Clarify what neuropsychological testing should explore without pre-interpreting results
- Use professional, accessible language in past tense flowing paragraphs
- Avoid bullet points
- Exclude diagnoses, differential diagnoses, recommendations, treatment planning, or objective test interpretation

These constraints align with the [[concepts/clinical-communication-register]] and [[concepts/narrative-report-generation]] principles documented elsewhere in the wiki.

## Key Design Features

- **Templated Variables**: Patient name, age, referral reason, and all history fields are injected via `{variable}` placeholders, making the prompt reusable across cases. This approach is consistent with [[concepts/neuropsychological-report-variables]] and [[concepts/neuropsychological-prompt-configuration]].
- **Role Prompting**: The model is instructed to write as a licensed neuropsychologist, grounding output in clinical voice and standards — a technique central to [[concepts/neuropsychological-prompt-engineering]].
- **Scope Limitation**: Strict exclusion of interpretive or diagnostic content preserves appropriate boundaries between the NSE narrative and later report sections, reflecting [[concepts/clinical-report-structure]] conventions.
- **PHI Awareness**: A dedicated preprocessing stage addresses redaction, supporting use in compliant clinical workflows via [[concepts/redaction-tokens]].

## Related Concepts
- [[concepts/clinical-nlp-pipelines]]
- [[concepts/behavioral-rating-scales]]
- [[concepts/validity-language]]

- [[concepts/neurobehavioral-status-exam]]
- [[concepts/neuropsychological-reporting]]
- [[concepts/staged-clinical-intake]]
- [[concepts/neuropsychological-prompt-engineering]]
- [[concepts/neuropsychological-report-variables]]
- [[concepts/clinical-report-structure]]
- [[concepts/phi-data-handling]]
- [[concepts/pii-redaction-pipelines]]
- [[concepts/narrative-report-generation]]
- [[concepts/clinical-communication-register]]