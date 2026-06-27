---
sources: [summaries/nt_interpretation.md, summaries/CLAUDE.md, summaries/report_body.md, summaries/NP-20240415-001_report.md, summaries/README.md, summaries/neuropsych-data-extractor.md, summaries/FIX_EXPLANATION.md, summaries/AS_PROCESSING_COMPLETE.md, summaries/AGENTS_luria.md]
brief: Numerical outputs of standardized cognitive tests, covering score types, norms, and long-format extraction schemas.
---

# Neuropsychological Test Scores & Score Types

Neuropsychological test scores are the numerical outputs of standardized cognitive assessments. Because raw performance on any single test is difficult to interpret in isolation, clinicians convert raw scores into several derived score types that situate a patient's performance relative to a normative reference group. Understanding these score types is essential for [[concepts/neuropsychological-reporting]] and for any automated extraction system such as those described in [[summaries/AGENTS_luria]] and [[summaries/neuropsych-data-extractor]].

## Core Score Types

### Raw Score
The unadjusted count of correct responses, errors, or time taken on a given task. Raw scores are test-specific and cannot be compared across instruments without conversion. In the cingulate long-format schema, this is captured in the `raw_score` column (NA if not reported).

### Scaled Score
- **Range**: Typically 1–19
- **Mean**: 10, **SD**: 3
- Used primarily within intelligence and memory batteries (e.g., WAIS-IV subtests)
- Allows comparison of subtest performance within a battery
- Mapped to `score_type = scaled_score` during automated extraction

### Standard Score
- **Range**: Broadly 40–160 in practice
- **Mean**: 100, **SD**: 15
- Used for composite and index scores (e.g., FSIQ, VCI, WMI, PSI)
- The most common metric for communicating overall cognitive ability levels
- Mapped to `score_type = standard_score`

### T-Score
- **Mean**: 50, **SD**: 10
- Common in personality, symptom validity, and some neuropsychological measures (e.g., MMPI, many executive function tests, behavioral rating scales)
- Positive T-scores above 65 or below 35 are typically flagged as clinically significant
- Mapped to `score_type = t_score`

### Z-Score
- **Mean**: 0, **SD**: 1
- Less common in clinical reporting but used in some neuropsychological batteries and research contexts
- Mapped to `score_type = z_score`

### Base Rate
- Expresses the frequency of a given score or score difference in the normative population
- Particularly relevant for discrepancy analysis and validity testing
- Captured as `score_type = base_rate`

### Percentile Rank
- Expresses what percentage of the normative sample a patient scored at or below
- Ranges from 0 to 99 (or 0.1 to 99.9 in some conventions)
- Highly intuitive for communicating results to non-specialist audiences
- Non-linear: small differences near the mean represent many percentile points; large score differences at extremes represent few
- Stored in the dedicated `percentile` column (separate from the `score` column) in the cingulate schema

### Composite & Index Scores
Several subtests are aggregated into **composite indices** that represent broad cognitive domains:
- **FSIQ** — Full Scale IQ
- **VCI** — Verbal Comprehension Index
- **PRI / VSI** — Perceptual Reasoning / Visual Spatial Index
- **WMI** — Working Memory Index
- **PSI** — Processing Speed Index

See [[concepts/cognitive-domains]] for the domain taxonomy these indices map onto.

## Qualitative Classifications

Numerical scores are typically paired with a qualitative descriptor band. The [[concepts/long-format-clinical-data]] schema stores these in a `range` column. The percentile-based thresholds used by the cingulate extraction pipeline are:

| Percentile | Qualitative Label |
|---|---|
| ≥ 91st | Above Average |
| 75–90 | High Average |
| 25–74 | Average |
| 9–24 | Low Average |
| 2–8 | Below Average / Borderline |
| < 2 | Impaired / Exceptionally Low |

For reference, standard-score equivalents follow this general convention:

| Standard Score Range | Classification |
|---|---|
| ≥ 130 | Extremely High / Very Superior |
| 120–129 | High Average / Superior |
| 110–119 | High Average |
| 90–109 | Average |
| 80–89 | Low Average |
| 70–79 | Borderline |
| 56–69 | Mildly Impaired |
| ≤ 55 | Moderately–Severely Impaired |

Classification labels vary slightly by test publisher but follow this general convention.

## Score-Type Mapping in Automated Extraction

A critical design decision in the cingulate ingestion pipeline (see [[summaries/neuropsych-data-extractor]]) is the use of **long format** rather than wide format. Each test/subtest measurement occupies **one row**, and the score type is stored in a `score_type` column. This means a single WAIS-IV administration that reports both a scaled score and a composite standard score generates multiple rows — one per measurement — rather than spreading values across parallel columns.

The extraction rules for `score_type` are:
- "scaled score" / "ss" / numeric range 1–19 → `scaled_score`
- "T-score" / mean 50 SD 10 convention → `t_score`
- "standard score" / mean 100 SD 15 convention → `standard_score`
- "z-score" → `z_score`
- base-rate values → `base_rate`

This schema is consumed directly by the [[concepts/cingulate-engine]] via DuckDB's `read_csv_auto` loader.

## Normative Comparisons

Scores are always interpreted relative to a **normative sample** — typically stratified by age, and sometimes by education or sex. The [[summaries/AGENTS_luria]] extraction schema captures this in the `normative_comparison` field (e.g., "Age 65–69 norms"), recognizing that the same raw score may be average for a 75-year-old but impaired for a 45-year-old.

## Discrepancy Analysis

A key clinical use of standardized scores is **discrepancy analysis** — comparing index scores to detect intra-individual strengths and weaknesses. A VCI–PSI discrepancy exceeding 20 standard-score points, for example, is flagged as a notable finding, as it may indicate neurological dysfunction, learning differences, or acquired injury.

## Multi-Rater Behavioral Scores

Behavioral rating scales (e.g., BRIEF, CBCL, BASC, BDEFS) add a further dimension: the same construct may be rated by multiple informants. The cingulate schema handles this by generating one row **per rater per scale**, with the `rater` column set to one of `self`, `parent`, `teacher`, or `examiner`. This prevents conflation of informant perspectives in downstream analytics.

## Domain and Subdomain Classification

Every score row in the cingulate schema is tagged with:
- `domain`: `Neuropsychological Test Score`, `Behavioral/Emotional/Social`, or `Effort/Validity Test`
- `subdomain`: e.g., "Working Memory", "Processing Speed", "Verbal Comprehension"
- `narrow`: narrow construct under the subdomain (e.g., "Auditory Working Memory", "Phonological Processing")
- `pass`: tag from [[concepts/pass-theory]] — Planning, Attention, Simultaneous, or Successive

This layered classification supports flexible grouping and filtering in the [[concepts/cingulate-engine]] and enables the domain-level summaries used in [[concepts/neuropsychological-reporting]].

## Related Pages
- [[concepts/neuropsychological-reporting]]
- [[concepts/cognitive-domains]]
- [[concepts/clinical-nlp-pipelines]]
- [[concepts/long-format-clinical-data]]
- [[concepts/cingulate-engine]]
- [[concepts/pass-theory]]
- [[concepts/pdf-score-extraction]]
- [[summaries/AGENTS_luria]]
- [[summaries/neuropsych-data-extractor]]

See also: [[summaries/AS_PROCESSING_COMPLETE]]

See also: [[summaries/FIX_EXPLANATION]]

See also: [[summaries/README]]

See also: [[summaries/NP-20240415-001_report]]

See also: [[summaries/report_body]]

See also: [[summaries/CLAUDE]]

See also: [[summaries/nt_interpretation]]