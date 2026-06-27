---
sources: [summaries/Luria_AI_Q4_Investor_Memo_2026.md, summaries/LLM Benchmark Comparison.md, summaries/NP-20240415-001_report.md, summaries/sirf_synthesis.md, summaries/nt_interpretation.md]
brief: Automated generation of structured clinical narrative text from quantitative scores using LLM prompting.
---

# Clinical Narrative Generation

Clinical narrative generation refers to the automated or semi-automated production of interpretive, prose-format clinical text from structured quantitative data — typically using a large language model (LLM) constrained by role, style, and output rules embedded in a prompt template.

## Core Concept

In neuropsychological practice, clinicians must translate rows of test scores into coherent, defensible written interpretations that communicate findings to referral sources, patients, and families. Clinical narrative generation automates this translation step by feeding structured score data into a carefully engineered prompt that instructs the model to behave as a domain expert and produce publication-ready prose.

The canonical example in this knowledge base is the `nt_interpretation` prompt template (see [[summaries/nt_interpretation]]), which casts the LLM as a licensed neuropsychologist and instructs it to produce a 2–4 paragraph domain-level interpretation from a set of `{score_lines}`.

## Key Design Elements

### Role Assignment
The prompt explicitly assigns a professional identity to the model ("licensed neuropsychologist"), anchoring vocabulary, tone, and clinical judgment thresholds. This is a foundational technique in [[concepts/neuropsychological-prompt-engineering]].

### Structured Input
Quantitative score data is passed as a variable (`{score_lines}`), providing the empirical grounding for the narrative. This connects clinical narrative generation to [[concepts/neuropsychological-test-scores]] and [[concepts/neuropsychological-score-interpretation]].

### Output Constraints
Effective clinical narrative generation specifies:
- **Format**: Flowing paragraphs, no bullet points
- **Tense and person**: Third-person past tense
- **Register**: Clinical but accessible (see [[concepts/clinical-communication-register]])
- **Structure**: Domain summary → score pattern → clinically significant findings

### Clinically Significant Thresholds
Prompts should instruct the model to flag scores at or below the **5th percentile** or at or above the **95th percentile**, consistent with standard [[concepts/neuropsychological-score-interpretation]] conventions.

## Relationship to Report Structure

Clinical narrative generation operates at the **domain level** within a larger [[concepts/clinical-report-structure]]. Each domain (e.g., memory, attention, executive function) receives its own generated narrative block, which is then assembled into a full report via [[concepts/modular-report-architecture]] or [[concepts/narrative-report-generation]] pipelines.

See also [[summaries/neuropsych-narrative-writer]] and [[summaries/nse_narrative]] for related implementations.

## Applications

- **Automated report drafting**: Reduces time from assessment to finalized report in neuropsychological practice
- **Standardization**: Promotes consistent language and structure across evaluators and time points
- **Training data generation**: Synthetic narratives can be used to train or fine-tune clinical NLP models (see [[concepts/clinical-nlp-pipelines]])
- **Quality assurance**: Generated narratives can be reviewed against score data as part of [[concepts/report-review-qa]]

## Related Concepts

- [[concepts/neuropsychological-prompt-engineering]]
- [[concepts/neuropsychological-reporting]]
- [[concepts/neuropsychological-assessment-automation]]
- [[concepts/cognitive-domains]]
- [[concepts/neuropsychological-report-variables]]
- [[concepts/neuropsychological-prompt-configuration]]
- [[concepts/domain-processor-pattern]]
- [[concepts/validity-language]]


See also: [[summaries/sirf_synthesis]]

See also: [[summaries/NP-20240415-001_report]]

See also: [[summaries/LLM Benchmark Comparison]]

See also: [[summaries/Luria_AI_Q4_Investor_Memo_2026]]