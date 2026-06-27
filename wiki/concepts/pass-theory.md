---
sources: [summaries/bwu.neuro.reports.recs.attention-problems.md, summaries/neuropsych-data-extractor.md]
brief: PASS Theory organizes cognition into Planning, Attention, Simultaneous, and Successive processes for assessment tagging.
---

# PASS Theory of Cognitive Processing

PASS Theory is a neuropsychologically grounded model of human cognitive processing developed by Das, Naglieri, and Kirby, rooted in A.R. Luria's functional units of the brain. It organizes cognition into four broad processes that underpin all intellectual activity and provides a principled taxonomy for tagging neuropsychological test scores in structured data pipelines.

## The Four PASS Processes

### Planning
Higher-order executive control: generating, selecting, and evaluating strategies to solve problems and regulate behavior. Maps onto prefrontal cortex function. Examples of tasks with Planning demands: Tower of London, Trail Making Test, verbal fluency.

### Attention
Selective and sustained allocation of cognitive resources while resisting distraction or interference. Maps onto the frontal-subcortical arousal system (Luria's Block 1/Block 3 overlap). Examples: Continuous Performance Tests, Stroop interference, Cancellation tasks.

### Simultaneous
Holistic, integrative processing where stimuli must be perceived or understood as a whole gestalt. Anchored in occipital-parietal function (Luria's Block 2, posterior). Examples: Block Design, Matrix Reasoning, figure copying, nonverbal analogies.

### Successive
Serial, sequential processing where stimuli form a chain-like order that must be maintained in strict sequence. Anchored in fronto-temporal function. Examples: Digit Span forward, sentence repetition, Word Series, serial arithmetic.

## Role in the Cingulate Neuropsych Pipeline

In the [[summaries/neuropsych-data-extractor]], the `pass` column in the [[concepts/long-format-clinical-data]] CSV schema carries one of four values — `Planning`, `Attention`, `Simultaneous`, or `Successive` — or `NA` when no PASS assignment is applicable. This tag enables downstream filtering and grouping within the [[concepts/cingulate-engine]] without requiring separate database tables per process.

The data extractor applies PASS tags at the row level (one tag per subtest row), allowing the same battery to be sliced by PASS dimension as an alternative to the standard domain/subdomain taxonomy. This is particularly useful when reporting frameworks organize profile interpretation around Luria-derived process distinctions.

## Relationship to Other Classification Axes

PASS tags operate alongside — not instead of — the `subdomain` and `narrow` columns in the schema. A single test row might carry:
- `subdomain`: "Working Memory"
- `narrow`: "Auditory Sequential Memory"
- `pass`: "Successive"

This layered structure allows clinicians and analysts to group scores by CHC broad abilities, PASS processes, or traditional cognitive domains depending on the referral question.

## Connection to the Luria Pipeline

The pipeline itself is named after Luria, whose three functional brain units directly inspired PASS Theory:

| Luria Unit | Function | PASS Process |
|---|---|---|
| Unit 1 (Brainstem/Limbic) | Arousal/Attention | Attention |
| Unit 2 (Occipital-Parietal-Temporal) | Coding/Processing | Simultaneous + Successive |
| Unit 3 (Prefrontal) | Programming/Regulation | Planning |

## Related Pages

- [[summaries/neuropsych-data-extractor]] — Schema and pipeline stage where the `pass` column is defined and populated
- [[concepts/neuropsychological-assessment-pipeline]] — Full ingestion pipeline context
- [[concepts/neuropsychological-test-scores]] — How individual scores are structured and typed
- [[concepts/cognitive-domains]] — Parallel classification axis using subdomain/narrow constructs
- [[concepts/long-format-clinical-data]] — Long-format schema that hosts the `pass` column
- [[concepts/neuropsychological-reporting]] — Downstream use of PASS-tagged data in clinical reports


See also: [[summaries/bwu.neuro.reports.recs.attention-problems]]