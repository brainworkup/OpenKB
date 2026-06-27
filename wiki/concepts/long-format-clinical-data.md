---
sources: [summaries/LLM_AGENT_MAP.md, summaries/CLAUDE.md, summaries/README.md, summaries/neuropsych-narrative-writer.md, summaries/neuropsych-data-extractor.md]
brief: Long-format schema storing one clinical measurement per row with a score_type discriminator column.
---

# Long-Format Clinical Data Schema

Long-format clinical data schema is a data-modeling convention in which **every individual measurement occupies exactly one row**, with a `score_type` discriminator column distinguishing scaled scores, T-scores, standard scores, z-scores, and base rates. This contrasts with wide format, where each score type becomes a separate column and a single test battery yields an unwieldy, sparse matrix.

The [[concepts/cingulate-engine]] depends on this schema exclusively. Its DuckDB loader (`read_csv_auto`) and downstream R processing utilities (notably `domain_processing_utils.R`) are built around long-format assumptions and will fail or produce incorrect results when given wide-format input.

## Why Long Format?

| Concern | Wide Format | Long Format |
|---|---|---|  
| Score-type variants | Separate columns (`scaled_score`, `t_score`, …) | Single `score_type` column |
| Sparse data | Many NA cells per row | Only rows that exist are written |
| Query patterns | Column-scan per score type | Filter on `score_type` value |
| Adding new score types | Schema change required | New value in `score_type` |
| DuckDB / SQL joins | Complex unpivot needed | Direct `GROUP BY domain, score_type` |

## Canonical Columns

As defined in the [[concepts/neuropsychological-assessment-pipeline]] and implemented by the [[summaries/neuropsych-data-extractor]]:

- **`test`** — Full test/subtest identifier (e.g. "WAIS-IV Digit Span")
- **`test_name`** — Short display label
- **`scale`** — Composite or index name (e.g. "Working Memory Index")
- **`raw_score`** — Numeric raw score or NA
- **`score`** — Numeric standardized score value
- **`score_type`** — Discriminator: `scaled_score`, `t_score`, `standard_score`, `z_score`, `base_rate`
- **`percentile`** — 0–99 or NA
- **`range`** — Qualitative classification ("Average", "Impaired", etc.)
- **`domain`** — One of: `Neuropsychological Test Score`, `Behavioral/Emotional/Social`, `Effort/Validity Test`
- **`subdomain`** — Cognitive subdomain (e.g. "Working Memory", "Processing Speed")
- **`narrow`** — Narrow construct under subdomain
- **`pass`** — [[concepts/pass-theory]] tag or NA
- **`verbal`** — `Verbal` or `Nonverbal`
- **`timed`** — `Timed` or `Untimed`
- **`description`** — One-sentence test description
- **`rater`** — `self`, `parent`, `teacher`, `examiner`
- **`age_group`** — `child`, `adolescent`, `adult`
- **`doc_id`** — Source document identifier
- **`date`** — Assessment date (YYYY-MM-DD)

**Required fields** (validated at `domain_processing_utils.R:991`): `domain`, `rater`, `age_group`, `test`.

## Score-Type Mapping

The [[summaries/neuropsych-data-extractor]] applies these mapping rules when parsing report text:

| Source Text Signals | `score_type` Value |
|---|---|
| "scaled score", "ss", range 1–19 | `scaled_score` |
| "T-score", mean 50 SD 10 | `t_score` |
| "standard score", mean 100 SD 15 | `standard_score` |
| "z-score" | `z_score` |
| "base rate", "BR" | `base_rate` |

## Multi-Rater Expansion

Behavioral rating instruments (BRIEF, CBCL, BASC, BDEFS) that collect data from multiple informants require **one row per rater per scale**. A parent-and-teacher BRIEF report thus yields two rows for each BRIEF subscale — identical except for the `rater` column. This keeps the schema normalized without a separate rater-pivot table.

## Range Classification

When a percentile is available, `range` is derived deterministically:

| Percentile | Range Label |
|---|---|
| ≥ 91 | Above Average |
| 75–90 | High Average |
| 25–74 | Average |
| 9–24 | Low Average |
| 2–8 | Below Average / Borderline |
| < 2 | Impaired / Exceptionally Low |

## PHI and Data Integrity

All values are preserved exactly as reported — no rounding, inference, or score-type conversion is permitted. [[concepts/phi-data-handling]] is handled upstream by the PDF parser stage; if any identifiers slip through they are replaced with `[PATIENT]`/`[CLINICIAN]` tokens (see [[concepts/pii-redaction-pipelines]]).

## Pipeline Context

Long-format CSV files are produced at stage 2 of the Luria ingestion pipeline and stored under `data-raw/csv/<doc_id>_neuropsych.csv`. They feed directly into the [[concepts/cingulate-engine]] for scoring, visualization, and eventual [[concepts/narrative-report-generation]].

```
pdf-parser → neuropsych-data-extractor → long-format CSV → cingulate engine → narrative
```

## Related Pages

- [[summaries/neuropsych-data-extractor]] — Stage 2 agent that writes this schema
- [[concepts/neuropsychological-assessment-pipeline]] — Full pipeline context
- [[concepts/neuropsychological-test-scores]] — Score types and norms
- [[concepts/cingulate-engine]] — Consuming engine (DuckDB-based)
- [[concepts/cognitive-domains]] — Domain/subdomain taxonomy
- [[concepts/pass-theory]] — PASS tagging column
- [[concepts/phi-data-handling]] — Privacy handling for clinical data
- [[concepts/clinical-data-management]] — Broader data management practices
- [[concepts/pdf-score-extraction]] — Upstream extraction step


See also: [[summaries/neuropsych-narrative-writer]]

See also: [[summaries/README]]

See also: [[summaries/CLAUDE]]

See also: [[summaries/LLM_AGENT_MAP]]