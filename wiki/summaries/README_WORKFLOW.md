---
doc_type: short
full_text: sources/README_WORKFLOW.md
---

# README_WORKFLOW: PAI RAG System Workflow Summary

## Overview
This document describes the end-to-end workflow for the PAI (Personality Assessment Inventory) [[concepts/retrieval-augmented-generation]] system, including knowledge base management, patient score entry, and automated clinical report generation.

## System Context
The [[concepts/pai-knowledge-base]] is built around a local collection of PAI-related documents:
- **79 PAI PDF reports** (`pai_00.pdf` through `pai_318.pdf`) in the `reports/` folder
- **19 research papers and PAI documentation** in the `source/` folder (prefixed `pai_source_`)
- **1 new patient file** (`AS_PAI_Report.pdf`) in the `input/` folder
- **Total:** 98 PDFs indexed in the knowledge base

## Key Workflow Steps

### Step 1: Rebuild the Knowledge Base
- Script: `rebuild_pai_ragnar.R`
- Removes old knowledge base, processes all 98 documents
- Builds a BM25 index (keyword search)
- Optionally generates vector embeddings via Ollama for semantic search
- Stores everything in `pai_knowledge_base.duckdb` (DuckDB backend, see [[concepts/duckdb-as-vector-store]])
- Estimated time: 5–10 minutes

### Step 2: Extract PAI T-Scores
- Source: `input/AS_PAI_Report.pdf` (pages 3–4, bar graph profiles)
- Target: `input/AS_scores_template.json` (pre-filled with patient demographics)
- Scores entered as T-scores per PAI scale (validity, clinical, treatment, interpersonal)
- Patient: Alessandra Snavely, age 36, female, 18 years education, tested 01/14/2026
- See also: [[concepts/pdf-score-extraction]] and [[concepts/neuropsychological-test-scores]]

### Step 3: Generate Interpretation Report
- Script: `generate_as_interpretation.R`
- Loads scores from JSON, queries the [[concepts/pai-knowledge-base]] per scale domain
- Produces AI-powered clinical interpretations via LLM
- Output saved to `output/AS_PAI_Interpretation_YYYYMMDD.txt`
- Estimated time: 2–5 minutes
- See also: [[concepts/neuropsychological-reporting]] and [[concepts/clinical-report-structure]]

## Core System Components
| File | Role |
|---|---|
| `pai_rag_system.R` | Core RAG query functions |
| `pai_score_interpreter.R` | PAI interpretation generator |
| `rebuild_pai_ragnar.R` | Knowledge base rebuild script |
| `generate_as_interpretation.R` | Patient-specific report generation |
| `pai_knowledge_base.duckdb` | Persistent knowledge base (DuckDB) |
| `WORKFLOW_INSTRUCTIONS.md` | Step-by-step operator guide |

## Search Architecture
- **BM25** (keyword-based): Always available, no external dependencies
- **Vector/semantic search**: Requires [[concepts/local-llm-inference]] (Ollama) to be running
- System is functional with BM25 alone; Ollama adds semantic retrieval depth via [[concepts/vector-search]]

## Notable Design Decisions
- Knowledge base rebuild was triggered by a file naming restructure (numbered system + `pai_source_` prefix)
- Multiple patients can be processed independently by creating separate JSON score files
- The [[concepts/pai-assessment]] pipeline is modular: score entry → RAG retrieval → LLM generation → formatted output
- Patient data handling follows [[concepts/phi-data-handling]] considerations given the presence of identifiable clinical information

## Related Concepts
- [[concepts/neuropsychological-assessment-pipeline]]
- [[concepts/clinical-data-privacy]]
- [[concepts/retrieval-augmented-generation]]
- [[concepts/pai-knowledge-base]]
- [[concepts/pai-assessment]]
- [[concepts/duckdb-as-vector-store]]
- [[concepts/vector-search]]
- [[concepts/local-llm-inference]]
- [[concepts/neuropsychological-reporting]]
- [[concepts/clinical-report-structure]]
- [[concepts/pdf-score-extraction]]
- [[concepts/neuropsychological-test-scores]]
- [[concepts/phi-data-handling]]