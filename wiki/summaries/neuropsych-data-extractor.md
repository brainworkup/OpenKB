---
doc_type: short
full_text: sources/neuropsych-data-extractor.md
---

# neuropsych-data-extractor

## Overview

The `neuropsych-data-extractor` is **stage 2** of the Luria neuropsych ingestion pipeline. It operates after the [[concepts/ocr-pipeline]] has produced clean, scrubbed text, and transforms that text into structured [[concepts/long-format-clinical-data]] rows suitable for direct consumption by the [[concepts/cingulate-engine]] via DuckDB's `read_csv_auto` loader. It does **not** perform narrative interpretation — that role belongs to the downstream narrative writer tool.

## Core Responsibility

Convert parsed neuropsychological report text into a **long-format CSV** where every test/subtest measurement occupies exactly one row. Score-type variants (scaled, T, standard, z, base-rate) are stored in a `score_type` column rather than spread across multiple columns (wide format is explicitly forbidden).

## Canonical Schema Columns

| Column | Purpose |
|---|---|
| `test` | Full test/subtest identifier (e.g. "WAIS-IV Digit Span") |
| `test_name` | Short label for tables |
| `scale` | Composite or index name (e.g. "Working Memory Index") |
| `raw_score` | Numeric raw score or NA |
| `score` | Numeric standardized score value |
| `score_type` | One of: `scaled_score`, `t_score`, `standard_score`, `z_score`, `base_rate` |
| `percentile` | 0–99 or NA |
| `range` | Qualitative classification (e.g. "Average", "Impaired") |
| `domain` | `Neuropsychological Test Score`, `Behavioral/Emotional/Social`, or `Effort/Validity Test` |
| `subdomain` | Cognitive subdomain (e.g. "Working Memory", "Processing Speed") |
| `narrow` | Narrow construct under subdomain |
| `pass` | PASS theory tag (`Planning`, `Attention`, `Simultaneous`, `Successive`) or NA |
| `verbal` | `Verbal` or `Nonverbal` |
| `timed` | `Timed` or `Untimed` |
| `description` | One-sentence test description |
| `rater` | `self`, `parent`, `teacher`, or `examiner` |
| `age_group` | `child`, `adolescent`, or `adult` |
| `doc_id` | Source document ID from the parser |
| `date` | Assessment date (YYYY-MM-DD) |

Required fields (checked by `domain_processing_utils.R:991`): `domain`, `rater`, `age_group`, `test`.

## Score-Type Mapping Rules

- "scaled score" / "ss" / range 1–19 → `scaled_score`
- "T-score" / mean 50, SD 10 → `t_score`
- "standard score" / mean 100, SD 15 → `standard_score`
- "z-score" → `z_score`

## Range Classification (Percentile Thresholds)

| Percentile | Label |
|---|---|
| ≥ 91st | Above Average |
| 75–90 | High Average |
| 25–74 | Average |
| 9–24 | Low Average |
| 2–8 | Below Average / Borderline |
| < 2 | Impaired / Exceptionally Low |

## Workflow

1. Receive parsed text + target output path (default: `data-raw/csv/<doc_id>_neuropsych.csv`).
2. Decompose each test into one row per subtest/score (e.g. WAIS-IV → ~20 rows).
3. Multi-rater scales (behavioral rating instruments such as BRIEF, CBCL, BASC, BDEFS) produce one row **per rater per scale**.
4. Map scores to `score_type`.
5. Classify `range` from percentile thresholds.
6. Write UTF-8 comma-separated CSV with quoted strings.
7. Verify output with `wc -l` and `head -3` shell commands.

## Output Summary Block

Final message must report:
- `CSV_PATH`, `ROW_COUNT`, `DOMAINS_DETECTED`, `RATERS`, `SCORE_TYPES`, and `WARNINGS`.

## Hard Rules

- [[concepts/long-format-clinical-data]] only — no wide-format columns.
- Preserve numeric values exactly; no rounding or conversion.
- Never invent tests, scores, or raters not present in source text.
- Replace any PHI that slipped through (names, IDs) with `[PATIENT]`/`[CLINICIAN]` and log a WARNING (see [[concepts/phi-data-handling]]).
- No interpretive language — that is the narrative report generation tool's role (see [[concepts/narrative-report-generation]]).

## Pipeline Position

```
pdf-parser → neuropsych-data-extractor → cingulate engine (DuckDB) → narrative-writer
```

## Related Concepts
- [[concepts/clinical-nlp-pipelines]]
- [[concepts/duckdb-as-vector-store]]
- [[concepts/pii-redaction-pipelines]]
- [[concepts/neuropsychological-reporting]]

- [[concepts/neuropsychological-assessment-pipeline]] — Overall ingestion pipeline architecture
- [[concepts/long-format-clinical-data]] — Long-format schema specification
- [[concepts/pdf-score-extraction]] — Extracting structured scores from PDF reports
- [[concepts/neuropsychological-test-scores]] — Standardized score types and classification
- [[concepts/pass-theory]] — Planning, Attention, Simultaneous, Successive cognitive model
- [[concepts/cognitive-domains]] — Domain/subdomain classification taxonomy
- [[concepts/phi-data-handling]] — PHI scrubbing and redaction in clinical pipelines
- [[concepts/cingulate-engine]] — Downstream DuckDB-based data processing engine