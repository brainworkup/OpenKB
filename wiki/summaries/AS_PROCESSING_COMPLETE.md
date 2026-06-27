---
doc_type: short
full_text: sources/AS_PROCESSING_COMPLETE.md
---

# AS Processing Complete — Demonstration Summary

**Date:** January 29, 2026
**Subject:** Alessandra Snavely (AS) PAI Processing
**Status:** Semantic RAG system demonstrated successfully with example scores

---

## Overview

This document records the successful demonstration of a [[concepts/retrieval-augmented-generation]] system built for processing [[concepts/pai-assessment]] (Personality Assessment Inventory) results. The system was tested using example T-scores for patient Alessandra Snavely; actual clinical processing awaits entry of real T-scores from the source PDF.

---

## System Specifications

| Parameter | Value |
|-----------|-------|
| Knowledge base size | 4,830 chunks from 98 documents |
| Embedding model | snowflake-arctic-embed2:568m |
| Search type | Semantic (vector similarity) |
| Search time | ~3–5 seconds per query |
| Similarity range | 0.51–0.79 |

---

## Key Capabilities Demonstrated

### Semantic Search vs. Keyword Search
- **Old method:** Exact keyword matching (e.g., `filter(str_detect(chunk_text, "validity"))`)
- **New method:** Vector similarity search using embeddings — finds conceptually relevant content even without exact keyword matches; see [[concepts/multimodal-embeddings]] for related embedding concepts
- Represents a major upgrade in retrieval quality for clinical interpretation

### Test Query Results
1. **Validity scale interpretation** — Similarity 0.575 from `pai_06.pdf` (NIM/PIM profile context)
2. **Normal clinical profile** — Similarity 0.786 from `pai_100.pdf` ("no marked elevations" context)
3. **Treatment planning** (SUI=56, STR=55) — Similarity 0.514 from `pai_00.pdf`

---

## System Improvement Over Previous Version

| Metric | Old System | New System | Change |
|--------|-----------|-----------|--------|
| Documents | 81 | 98 | +21% |
| Chunks | 2,546 | 4,830 | +90% |
| Search method | Keyword | Semantic | Major upgrade |

---

## Files Created

- `process_AS_complete.R` — Interactive T-score entry and full processing
- `demo_AS_with_example_scores.R` — Demonstration script (completed)
- `README_AS_PROCESSING.md` — Usage instructions
- `input/AS_scores_template.json` — Score entry template
- `output/AS_Demo_Output_*.txt` — Demo output files

---

## Processing Options for Real T-Scores

Three methods available:
1. **Interactive R script** — Prompts for each T-score, saves to JSON, generates interpretation
2. **Pre-filled script** — Edit example scores in `demo_AS_with_example_scores.R`
3. **Manual JSON entry** — Edit `input/AS_scores_template.json` directly, then run processor

Source data: `input/AS_PAI_Report.pdf` pages 3–4 (bar graphs)

---

## Clinical Context

- **Patient:** Alessandra Snavely, 36-year-old female, 18 years education
- **Test date:** 01/14/2026
- **Example validity scales used in demo:** ICN=45, INF=48, NIM=44, PIM=50 (all <60, profile valid)
- System retrieves evidence-based interpretive contexts from the 98-document [[concepts/pai-knowledge-base]] and feeds them to a local LLM for interpretation generation; see [[concepts/local-llm-inference]] for related infrastructure

---

## Next Steps

1. Open `AS_PAI_Report.pdf` and read actual T-scores from pages 3–4
2. Run `process_AS_complete.R` (interactive) or edit the JSON template
3. Review generated interpretation against retrieved clinical contexts
4. Apply clinical judgment to finalize report

---

## Related Concepts
- [[concepts/sqlite-as-vector-store]]
- [[concepts/clinical-data-privacy]]
- [[concepts/clinical-report-structure]]
- [[concepts/neuropsychological-test-scores]]

- [[concepts/retrieval-augmented-generation]] — Architecture underlying this system
- [[concepts/pai-assessment]] — The psychological instrument being interpreted
- [[concepts/pai-knowledge-base]] — The 98-document corpus used for evidence retrieval
- [[concepts/local-llm-inference]] — Local model infrastructure supporting the pipeline
- [[concepts/neuropsychological-assessment-pipeline]] — Broader pipeline context for this workflow
- [[concepts/neuropsychological-reporting]] — Clinical output goals of the system
- [[concepts/phi-data-handling]] — Relevant given patient data processed through the system