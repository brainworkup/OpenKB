---
sources: [summaries/LLM Benchmark Comparison.md, summaries/Apply-to-Y-Combinator-JWT.md, summaries/redesign_20260623110910.md, summaries/redesign_20260623110817.md, summaries/sirf_synthesis.md]
brief: The integrative section of a neuropsychological report that unifies test data, observations, and clinical context.
---

# Neuropsychological Synthesis and Clinical Impressions

The **Synthesis and Clinical Impressions** section is the culminating interpretive component of a formal neuropsychological evaluation report. It moves beyond raw scores and isolated observations to produce a unified, clinically meaningful narrative that addresses the referral question and guides diagnostic and treatment decisions.

## Purpose and Function

Unlike earlier report sections that present data domain by domain, the synthesis integrates:
- **Behavioral and clinical observations** from the neurobehavioral status exam
- **Objective test performance** across cognitive domains
- **Clinically flagged scores** (typically ≤16th or ≥84th percentile, representing ~1 SD deviation)
- **Real-world functional impact** of identified strengths and weaknesses

The result is a 3–5 paragraph narrative written in professional, past-tense, DSM-5-TR aligned clinical language.

## Core Synthesis Tasks

### 1. Integration of Qualitative and Quantitative Data
The synthesis bridges the clinician's direct observations (from the [[concepts/neurobehavioral-status-exam]]) with performance on standardized [[concepts/neuropsychological-tests]]. A patient may perform adequately on isolated subtests yet show meaningful impairment when scores are viewed as a pattern across [[concepts/cognitive-domains]].

### 2. Identification of Strengths and Weaknesses
The synthesis explicitly characterizes the patient's cognitive profile — noting both preserved abilities and areas of relative deficit. This dual framing supports nuanced interpretation rather than a deficit-only focus. See [[concepts/neuropsychological-score-interpretation]] for threshold conventions.

### 3. Functional and Real-World Impact
Findings are translated into everyday terms: how do identified weaknesses affect occupational performance, independent living, academic functioning, or interpersonal relationships? This anchors the report in practical clinical utility.

### 4. Referral Question Contextualization
Every synthesis must loop back to why the evaluation was requested. Whether the question involves differential diagnosis, disability determination, treatment planning, or forensic purposes (see [[concepts/forensic-neuropsychological-evaluation]]), the synthesis must directly address it.

### 5. Clinical Language Standards
The synthesis uses DSM-5-TR aligned terminology, avoids colloquialisms, and is written in flowing past-tense paragraphs — conventions shared across [[concepts/clinical-narrative-generation]] and [[concepts/clinical-communication-register]].

## Flagged Score Thresholds

| Threshold | Interpretation |
|---|---|
| ≤16th percentile | Area of weakness (~1 SD below mean) |
| ≥84th percentile | Area of strength (~1 SD above mean) |

These cutoffs are consistent with standard neuropsychological practice and align with [[concepts/neuropsychological-score-interpretation]].

## Automation and Prompt Engineering

The synthesis section is a prime target for LLM-assisted report generation. The [[summaries/sirf_synthesis]] document provides a structured prompt template that injects patient-specific data (NSE summary, test narrative, flagged scores) into a role-prompted LLM to produce synthesis-quality output. This approach is part of the broader [[concepts/neuropsychological-assessment-automation]] and [[concepts/neuropsychological-prompt-engineering]] ecosystems.

Related automation components include:
- [[concepts/narrative-report-generation]] — general LLM narrative output pipelines
- [[concepts/modular-report-architecture]] — how synthesis fits within a larger report structure
- [[concepts/clinical-report-structure]] — section sequencing in formal reports
- [[concepts/neuropsychological-reporting]] — overarching reporting conventions

## Related Concepts
- [[concepts/neuropsychological-assessment-pipeline]]
- [[concepts/cognitive-domains]]
- [[concepts/executive-function-deficits]]
- [[concepts/working-memory]]
- [[concepts/validity-language]]
- [[concepts/neuropsychological-prompt-configuration]]
- [[concepts/neuropsychological-report-variables]]
- [[summaries/nse_narrative]]
- [[summaries/nt_interpretation]]
- [[summaries/neuropsych-narrative-writer]]


See also: [[summaries/redesign_20260623110817]]

See also: [[summaries/redesign_20260623110910]]

See also: [[summaries/Apply-to-Y-Combinator-JWT]]

See also: [[summaries/LLM Benchmark Comparison]]