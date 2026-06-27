---
sources: [summaries/bwu.neuro.reports.recs.adhd.dref-intervention-strategies.md, summaries/CARS2-Manual_extracted.md, summaries/pai_02.md, summaries/pai_01.md, summaries/pai_00.md, summaries/NP-20240415-001_report.md, summaries/sirf_synthesis.md, summaries/nt_interpretation.md]
brief: Translating neuropsychological test scores into clinically meaningful cognitive statements via normative and pattern analysis.
---

# Neuropsychological Score Interpretation

Neuropsychological score interpretation is the process of translating quantitative test performance — raw scores, scaled scores, standard scores, and percentile ranks — into clinically meaningful statements about a patient's cognitive functioning. It forms the analytical core of any neuropsychological evaluation report and requires integration of normative data, domain knowledge, and pattern recognition across multiple measures.

## Core Principles

### Normative Anchoring
Scores are interpreted relative to age- (and often education-) matched normative samples. The most clinically meaningful metric is typically the **percentile rank**, which describes where a patient falls relative to peers. Key threshold conventions include:

- **≤ 5th percentile** — Clinically significant low performance; warrants explicit flagging in the narrative
- **≥ 95th percentile** — Clinically significant high performance; may indicate exceptional strength or ceiling effects
- **16th–84th percentile** — Broadly average range (±1 SD for normally distributed scores)

These thresholds are directly encoded in the [[summaries/nt_interpretation]] prompt template, which instructs the model to highlight any scores meeting either criterion.

### Consistency Between Classification Labels and Percentile Values
A recurring interpretive hazard is the **mislabeling of scores** when classification labels and percentile ranks are reported independently. The case NP-20240415-001 illustrates this clearly: the WAIS-IV Digit Span subtest yielded a scaled score of 9 (37th percentile), which falls squarely within the Average classification band, yet the report simultaneously flagged it as "clinically below average (≤16th percentile)." This internal inconsistency — a scaled score of 9 appearing with a below-average clinical flag — highlights the need for automated cross-validation of label, scaled score, and percentile rank when generating reports. Any pipeline producing [[concepts/neuropsychological-reporting]] output should enforce consistency checks between these three fields.

### Domain-Level Summarization
Interpretation proceeds in two layers:

1. **Domain-level summary** — An overall characterization of functioning (e.g., "fell in the low average range") anchors the narrative and orients the reader before granular detail is provided.
2. **Subtest/scale-level pattern** — Individual scores within the domain are described to reveal intra-domain strengths and weaknesses, which often carry greater diagnostic value than composite scores alone.

This two-layer structure mirrors the output format specified in [[summaries/nt_interpretation]] and is a standard convention in [[concepts/neuropsychological-reporting]].

### Intra-Domain Variability
A critical interpretive task is identifying meaningful **scatter** — significant discrepancies among subtests within the same cognitive domain. Scatter may indicate:
- Process-specific deficits (e.g., encoding versus retrieval dissociations in memory)
- Modality-specific effects (verbal vs. visual)
- Effort or validity concerns (see [[concepts/validity-language]])

When a single subtest is used to characterize an entire domain — as occurred in the NP-20240415-001 evaluation, where Digit Span alone was mapped to working memory, attention, executive function, memory, language, and visuospatial ability — scatter analysis is impossible and interpretive validity is severely constrained. Reports should explicitly acknowledge such limitations.

## Relationship to Score Types

Different score metrics require different interpretive framing. Common metrics encountered in neuropsychological evaluation include:

| Score Type | Typical Mean | SD | Notes |
|---|---|---|---|
| Standard Score (SS) | 100 | 15 | Used for composites |
| Scaled Score | 10 | 3 | Used for subtests |
| T-Score | 50 | 10 | Common in rating scales |
| Percentile Rank | 50th | — | Nonparametric; most interpretable |

For structured data about these score types, see [[concepts/neuropsychological-test-scores]] and [[concepts/neuropsychological-tests]].

### Working Memory as an Illustrative Domain
The [[concepts/working-memory-index]] from the [[concepts/wais-iv]] provides a well-documented example of score interpretation in practice. The Digit Span subtest (scaled score mean = 10, SD = 3) contributes to this composite. A scaled score of 9 places a patient at the 37th percentile — solidly Average — yet clinical concern may be warranted in context (e.g., a suspected diagnosis of [[concepts/mild-cognitive-impairment]]). This illustrates that normative classification alone is insufficient; diagnostic context, referral question, and longitudinal change all inform the final interpretive statement.

## Scope Limitations and Report Validity

Interpretive validity depends on the breadth of the assessment battery. A comprehensive neuropsychological evaluation typically spans multiple instruments across all major [[concepts/cognitive-domains]]. When evaluation is restricted to a single subtest:

- Cross-domain comparisons are not possible
- Intra-domain scatter cannot be assessed
- Composite index scores cannot be computed
- Diagnostic conclusions should be explicitly qualified

Background information (age, education, medical history, referral reason) is also essential for contextualizing scores. Its absence, as in the NP-20240415-001 report, represents a significant documentation gap that limits interpretive depth. These considerations are formalized in the caveats and limitations sections of [[concepts/clinical-report-structure]].

## Automated Interpretation

The [[summaries/nt_interpretation]] prompt template automates domain narrative generation by:
- Accepting structured `{score_lines}` as input
- Applying the interpretive conventions described above
- Producing flowing prose in third-person past tense
- Flagging clinically significant percentile values automatically

This places score interpretation within the broader workflow of [[concepts/neuropsychological-assessment-automation]] and [[concepts/clinical-narrative-generation]]. The template-based approach supports standardization across evaluators and integrates naturally into [[concepts/modular-report-architecture]] pipelines. Importantly, automated pipelines must also implement **cross-field consistency validation** to avoid the classification-percentile mismatches noted above.

## Related Concepts

- [[concepts/neuropsychological-reporting]] — The broader report structure that contextualizes score interpretation
- [[concepts/cognitive-domains]] — The domains across which scores are organized and interpreted
- [[concepts/neuropsychological-prompt-engineering]] — How prompts encode interpretive conventions for LLM use
- [[concepts/neuropsychological-report-variables]] — The variable schema used to pass scores into templates
- [[concepts/clinical-report-structure]] — How interpretation sections fit into the full report
- [[concepts/validity-language]] — Language conventions for describing performance validity alongside scores
- [[concepts/neuropsychological-assessment-pipeline]] — End-to-end pipeline in which score interpretation is a stage
- [[concepts/wais-iv]] — A primary source of scaled scores and composite indices in adult neuropsychological evaluation
- [[concepts/working-memory-index]] — A composite index illustrating multi-subtest score aggregation
- [[concepts/mild-cognitive-impairment]] — A clinical context in which score interpretation must be carefully calibrated
- [[summaries/nt_interpretation]] — Prompt template encoding interpretive conventions for LLM use
- [[summaries/neurocog.prompt]] — A related prompt template for cognitive domain narratives
- [[summaries/nse_narrative]] — Example narrative output demonstrating these interpretive conventions
- [[summaries/NP-20240415-001_report]] — A case example illustrating classification-percentile inconsistency and scope limitations

See also: [[summaries/sirf_synthesis]]

See also: [[summaries/pai_00]]

See also: [[summaries/pai_01]]

See also: [[summaries/pai_02]]

See also: [[summaries/CARS2-Manual_extracted]]

See also: [[summaries/bwu.neuro.reports.recs.adhd.dref-intervention-strategies]]