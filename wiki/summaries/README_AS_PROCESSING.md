---
doc_type: short
full_text: sources/README_AS_PROCESSING.md
---

# README: AS PAI Report Processing

## Overview

This README describes a workflow for processing a [[concepts/pai-assessment|PAI (Personality Assessment Inventory)]] report for a specific patient (Alessandra Snavely, 36F, tested 01/14/2026). It provides two usage modes and documents the full pipeline from score entry to report generation using [[concepts/retrieval-augmented-generation|semantic RAG]] retrieval.

## Two Processing Modes

### Option 1: Interactive
- Run `process_AS_complete.R` in R
- Script prompts for T-score entry interactively

### Option 2: Manual Score Entry
- Read T-scores from PDF (`input/AS_PAI_Report.pdf`, pages 3–4)
- Edit `input/AS_scores_template.json` with actual T-scores
- Run same R script

## Pipeline Steps

1. **Load patient info** — Name, age, sex, test date
2. **Collect/validate T-scores** — Interactive or template-based; saves to JSON
3. **Identify elevated scales** — Flags T-scores ≥ 60; summarizes clinical profile
4. **[[concepts/retrieval-augmented-generation|Semantic RAG search]]** — Queries a [[concepts/pai-knowledge-base|98-document knowledge base]] (4,830 chunks) using [[concepts/vector-search|embeddings]] for four domains:
   - Validity interpretation
   - Clinical profile
   - Treatment considerations
   - Interpersonal functioning
5. **Generate report** — Outputs contextualized text ready for LLM interpretation

## Key Technical Details

- **Language/Environment:** R 4.5.2
- **Embedding model:** `snowflake-arctic-embed2:568m` via Ollama
- **Key R packages:** tidyverse, arrow, ragnar, jsonlite
- **Knowledge base:** 98 documents, 4,830 embedded chunks
- **Working directory:** `/Users/joey/rag/pai`

## Input/Output Files

| Role | File |
|------|------|
| Input PDF | `input/AS_PAI_Report.pdf` |
| Score template | `input/AS_scores_template.json` |
| Full data output | `output/AS_PAI_semantic_interpretation_YYYYMMDD.rds` |
| Clinical report | `output/AS_PAI_Report_YYYYMMDD.txt` |

## Estimated Runtime

- Score entry: 2–5 minutes
- Semantic search: 2–3 minutes
- Report generation: Instant
- **Total: ~5–10 minutes**

## Troubleshooting

- **Ollama not running:** `ollama serve`
- **Model missing:** `ollama pull snowflake-arctic-embed2:568m`
- **Embeddings missing:** Ensure embedding generation was run from `/Users/joey/rag/pai`

## Related Concepts
- [[concepts/vector-search]]

- [[concepts/pai-assessment]] — The PAI instrument and T-score interpretation
- [[concepts/pai-knowledge-base]] — The 98-document PAI literature knowledge base
- [[concepts/retrieval-augmented-generation]] — Retrieval-Augmented Generation using embeddings
- [[concepts/clinical-report-structure]] — LLM-assisted psychological report writing
- [[concepts/neuropsychological-assessment-pipeline]] — Broader clinical assessment workflows
- [[concepts/pdf-score-extraction]] — Extracting scores from PDF reports
- [[concepts/local-llm-inference]] — Running Ollama models locally for embedding queries
- [[concepts/parquet-as-knowledge-store]] — Arrow/Parquet-based storage for embedded chunks