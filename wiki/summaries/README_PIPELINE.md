---
doc_type: short
full_text: sources/README_PIPELINE.md
---

# README: PAI Interpretation Pipeline

## Overview

This document describes an AI-powered system for generating comprehensive clinical interpretations of [[concepts/pai-assessment]] (Personality Assessment Inventory) scores. The pipeline combines manual T-score input with a [[concepts/retrieval-augmented-generation]] knowledge base to produce structured clinical reports via large language models.

---

## Purpose and Workflow

The system bridges the gap between raw PAI T-scores (which are often embedded in graphical PDF charts and not machine-readable) and professional clinical narrative reports. The pipeline consists of:

1. **Manual score entry** — User inputs T-scores from PAI PDF charts (validity, clinical, treatment, and interpersonal scales)
2. **RAG knowledge base query** — Scores are used to retrieve relevant interpretation content from a [[concepts/duckdb-as-vector-store]]-backed knowledge base
3. **LLM-powered interpretation** — A language model generates narrative sections for each domain
4. **Structured report output** — A formatted clinical report is produced as a text file

---

## Key Components

### Core R Files
- **`interpret_pai_from_scores.R`** — Primary user-facing interface for score entry and pipeline execution
- **`pai_rag_system.R`** — [[concepts/retrieval-augmented-generation]] search functions and LLM integration
- **`pai_complete_pipeline.R`** — Interpretation generation and report formatting logic; see also [[concepts/modular-report-architecture]]
- **`rebuild_pai_ragnar.R`** — Knowledge base construction from source PDFs
- **`pai_knowledge_base_copy.duckdb`** — DuckDB vector knowledge store (see [[concepts/pai-knowledge-base]])

### LLM Provider Options
- **Ollama** (local inference via [[concepts/local-llm-inference]], e.g., `llama3.2`)
- **OpenAI** (e.g., `gpt-4o-mini`; see [[concepts/openai-compatible-api]])
- **Anthropic** (e.g., `claude-3-5-sonnet-20241022`)

---

## PAI Scales Covered

### Validity Scales
- ICN (Inconsistency), INF (Infrequency), NIM (Negative Impression), PIM (Positive Impression)

### Clinical Scales
- SOM, ANX, ARD, DEP, MAN, PAR, SCZ, BOR, ANT, ALC, DRG

### Treatment Scales
- AGG, SUI, STR, NON, RXR

### Interpersonal Scales
- DOM, WRM

### Subscales (Optional)
- E.g., SOM-C, SOM-S, SOM-H, DEP-C, DEP-A, DEP-P

---

## Generated Report Structure

1. Patient Information
2. Validity of Test Results
3. Clinical Profile
4. Subscale Analysis
5. Treatment Considerations
6. Interpersonal Functioning
7. Integrated Summary

See [[concepts/clinical-report-structure]] and [[concepts/neuropsychological-reporting]] for related discussion of report organization conventions.

---

## Technical Notes

- **Why manual input?** PAI PDFs use graphical bar charts for T-scores; OCR cannot reliably extract these values. See [[concepts/pdf-score-extraction]] and [[concepts/ocr-pipeline]] for related challenges.
- **RAG parameters**: Configurable `top_k` (number of source chunks), `semantic_weight`, and `text_weight` for retrieval tuning; see [[concepts/vector-search]].
- **Knowledge base**: Built from PDFs using `rebuild_pai_ragnar.R` and stored in DuckDB; see [[concepts/pai-knowledge-base]] and [[concepts/duckdb-as-vector-store]].

---

## Clinical Disclaimers

- Reports are **AI-assisted**, not a replacement for professional clinical judgment.
- Interpretations must be cross-checked with clinical interview, collateral data, and other assessments.
- Output quality is directly dependent on the accuracy of manual score entry.
- Handling of patient data should follow [[concepts/phi-data-handling]] and [[concepts/clinical-data-privacy]] practices.

---

## Related Concepts
- [[concepts/neuropsychological-assessment-pipeline]]
- [[concepts/pai-assessment]]
- [[concepts/pai-knowledge-base]]
- [[concepts/retrieval-augmented-generation]]
- [[concepts/local-llm-inference]]
- [[concepts/clinical-report-structure]]
- [[concepts/neuropsychological-reporting]]
- [[concepts/pdf-score-extraction]]
- [[concepts/duckdb-as-vector-store]]
- [[concepts/clinical-nlp-pipelines]]

---

**Last Updated**: January 29, 2026