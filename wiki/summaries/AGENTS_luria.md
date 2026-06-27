---
doc_type: short
full_text: sources/AGENTS_luria.md
---

# Sheets Data Indexer Worker (AGENTS_luria)

## Overview

This document specifies the **Sheets Data Indexer Worker**, the third stage of a [[concepts/neuropsychological-assessment-pipeline]]. Its primary responsibility is to persist structured JSON records — produced by the upstream Neuropsych Data Extractor — into local storage systems (SQLite and LanceDB), forming the searchable knowledge index for a [[concepts/retrieval-augmented-generation]] system.

## Role in the Pipeline

| Stage | Role |
|-------|------|
| Stage 1 | PDF parsing / text extraction |
| Stage 2 | Neuropsych Data Extractor (structured JSON output) |
| **Stage 3** | **Sheets Data Indexer Worker (this document)** |

This worker is invoked **after** the Neuropsych Data Extractor has produced structured JSON data records.

## Storage Architecture

### SQLite — Structured Relational Storage

Two primary tables are maintained:

#### TestScores Table
Stores granular psychometric data including:
- **Document metadata**: `doc_id`, `doc_type`, `date`, `examiner`, `referral_reason`
- **Test data**: `test_name`, `subtest_name`, `raw_score`, `scaled_score`, `standard_score`, `t_score`, `percentile_rank`, `classification`, `composite_index`
- **Clinical context**: `normative_comparison`, `primary_diagnosis`, `clinical_impression`, `cognitive_domains_affected`, `recommendations`, `notable_findings`

Score types follow standard [[concepts/neuropsychological-test-scores]] conventions:
- Scaled scores: 1–19 range
- Standard scores: 100 ± 15
- T-scores: 50 ± 10
- Percentile ranks: 0–99

#### ClinicalSummaries Table
One row per document (`doc_id`), aggregating high-level clinical information: diagnosis, clinical impression, affected [[concepts/cognitive-domains]], recommendations, and notable findings.

### LanceDB — Vector Store for Semantic Search

Narrative text is chunked by **section headers** and embedded using `sentence-transformers/all-MiniLM-L6-v2` (384-dimensional vectors). Each chunk is stored in LanceDB with `doc_id` and `chunk_id` metadata, enabling semantic search over clinical narratives as part of the [[concepts/retrieval-augmented-generation]] index.

## Workflow Steps

1. **Register document** — idempotent upsert of document identity.
2. **Append test-score rows** — skip insertion if `doc_id` already exists (no overwrite).
3. **Upsert clinical summary** — one row per `doc_id` in ClinicalSummaries.
4. **Index narrative chunks** — embed and store text segments in LanceDB.

## Key Guidelines

- **Immutability**: Never overwrite existing rows; only append new data.
- **Data fidelity**: Preserve exact values from the extractor — no rounding or reformatting.
- **Missing data**: Use `"N/A"` for any empty or missing fields.
- **Date format**: All dates must conform to `YYYY-MM-DD`.

## Concepts Referenced

- [[concepts/retrieval-augmented-generation]] — RAG system this index supports.
- [[concepts/neuropsychological-assessment-pipeline]] — Full pipeline context in which this worker operates.
- [[concepts/neuropsychological-test-scores]] — Scoring conventions (scaled, standard, T-scores).
- [[concepts/cognitive-domains]] — Cognitive domain classifications stored in both tables.
- [[concepts/phi-data-handling]] — Relevant to the handling of sensitive neuropsychological records.
- [[concepts/clinical-nlp-pipelines]] — Broader context for NLP-driven clinical data processing.

## Related Concepts
- [[concepts/neuropsychological-reporting]]
- [[concepts/persistent-memory]]
