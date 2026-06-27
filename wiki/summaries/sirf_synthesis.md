---
doc_type: short
full_text: sources/sirf_synthesis.md
---

# SIRF Synthesis Prompt Template

## Overview
This document is a **structured prompt template** designed for use with a large language model acting as a licensed neuropsychologist. Its purpose is to generate the *Synthesis and Clinical Impressions* section of a formal neuropsychological evaluation report.

## Inputs (Template Variables)
The prompt accepts four injectable variables:
- `{patient_name}` — Identifier for the patient being evaluated
- `{nse_summary}` — Summary of clinical observations from the Neurological Status Examination (NSE)
- `{nt_narrative}` — Narrative description of neuropsychological test performance
- `{flagged_scores}` — Clinically significant scores (at or below the 16th percentile or at/above the 84th percentile)

## Output Requirements
The template instructs the LLM to produce a **3–5 paragraph synthesis** that:
1. Integrates NSE clinical observations with objective test performance patterns
2. Identifies overarching cognitive **strengths** and **areas of relative weakness** across domains
3. Describes **functional and real-world impact** of identified deficits
4. Contextualizes findings relative to the **referral question**
5. Uses professional, DSM-5-TR aligned language (see [[concepts/clinical-communication-register]]) in flowing past-tense paragraphs

## Clinical and Technical Context
- The flagged score thresholds (≤16th or ≥84th percentile) align with standard neuropsychological cutoffs for clinically meaningful deviation (~1 SD from the mean), consistent with practices described in [[concepts/neuropsychological-score-interpretation]]
- The synthesis integrates qualitative (NSE observations) with quantitative (test scores) data — a core practice in [[concepts/neuropsychological-assessment-pipeline]]
- Output is intended to be inserted directly into a formal evaluation report as part of a [[concepts/modular-report-architecture]]
- The prompt role-plays the LLM as a licensed neuropsychologist, which shapes tone, vocabulary, and interpretive framing — an instance of [[concepts/neuropsychological-prompt-engineering]]

## Use Cases
- Automated or semi-automated neuropsychological report generation via [[concepts/neuropsychological-assessment-automation]]
- Standardizing synthesis quality across evaluators as part of [[concepts/clinical-report-structure]]
- Training or quality assurance for report writing

## Related Concepts
- [[concepts/neurobehavioral-status-exam]]
- [[concepts/neuropsychological-synthesis]]
- [[concepts/cognitive-domains]]
- [[concepts/clinical-narrative-generation]]
- [[concepts/neuropsychological-reporting]]
- [[concepts/neuropsychological-prompt-configuration]]
- [[concepts/narrative-report-generation]]
- [[concepts/validity-language]]