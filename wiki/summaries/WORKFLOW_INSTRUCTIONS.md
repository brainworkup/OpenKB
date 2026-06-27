---
doc_type: short
full_text: sources/WORKFLOW_INSTRUCTIONS.md
---

# WORKFLOW_INSTRUCTIONS — PAI RAG System Complete Workflow

**Date:** January 29, 2026
**Patient:** Alessandra Snavely (AS), Age 36, Female, 18 years education, Single
**Test Date:** 01/14/2026

## Overview

This document describes a three-step automated workflow for generating clinical interpretations of [[concepts/pai-assessment]] (Personality Assessment Inventory) scores using a [[concepts/retrieval-augmented-generation]] (RAG) knowledge base built with the R `ragnar` package.

## Workflow Steps

### Step 1: Rebuild Knowledge Base

- Script: `rebuild_pai_ragnar.R`
- Sources: **79 PAI reports** (`pai_00.pdf` – `pai_318.pdf`) + **19 research/source documents** (`pai_source_` prefix) = 98 PDF files total
- Process: removes old KB → imports PDFs → chunks text → builds [[concepts/hybrid-search-retrieval]] index → generates embeddings (via [[concepts/local-llm-inference]] if available) → saves to `pai_knowledge_base.duckdb`
- Expected runtime: 5–10 minutes

### Step 2: Extract PAI T-Scores for New Patient

Two options:
- **Option A:** Use a Summary Table PDF (e.g., `AS_SummaryTable.pdf`) if available
- **Option B (most common):** Manually read T-scores from bar graphs on pages 3–4 of the PAI report and populate `input/AS_scores_template.json`

#### PAI Scale Categories Covered

| Category | Scales |
|---|---|
| **Validity** | ICN, INF, NIM, PIM |
| **Clinical** | SOM, ANX, ARD, DEP, MAN, PAR, SCZ, BOR, ANT, ALC, DRG |
| **Treatment** | AGG, SUI, STR, NON, RXR |
| **Interpersonal** | DOM, WRM |

### Step 3: Generate Interpretation Report

- Script: `generate_as_interpretation.R`
- Process: loads scores → queries RAG knowledge base per scale domain → generates AI clinical interpretations → formats report → saves to `output/AS_PAI_Interpretation_YYYYMMDD.txt`

## Key Files

| File | Purpose |
|---|---|
| `rebuild_pai_ragnar.R` | Rebuilds DuckDB knowledge base |
| `AS_scores_template.json` | Manual T-score entry template |
| `generate_as_interpretation.R` | Generates interpretation report |
| `process_new_patient_complete.R` | Full automated workflow |
| `pai_knowledge_base.duckdb` | Persistent vector/BM25 search store |

## Technical Notes

- Uses R package `ragnar` for the [[concepts/retrieval-augmented-generation]] pipeline
- Falls back to BM25 keyword search (see [[concepts/fallback-strategy]]) if Ollama embeddings are unavailable
- The DuckDB store functions as a [[concepts/duckdb-as-vector-store]] for persistent retrieval
- T-scores extracted from [[concepts/clinical-pdf-assessment]] reports feed into the pipeline
- Previous example patient: Christi McD (`McD, C_SummaryTable.pdf`)
- Output can be exported to Word, PDF, or archived in patient records
- Patient data handling follows [[concepts/phi-data-handling]] considerations for clinical records

## Post-Report Steps

1. Review and validate AI-generated interpretation
2. Add clinician notes
3. Export to preferred format
4. Archive in patient records system

## Related Concepts
- [[concepts/pai-knowledge-base]]
- [[concepts/neuropsychological-assessment-pipeline]]
- [[concepts/pdf-score-extraction]]
- [[concepts/clinical-data-privacy]]
