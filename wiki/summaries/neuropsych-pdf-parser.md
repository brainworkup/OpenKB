---
doc_type: short
full_text: sources/neuropsych-pdf-parser.md
---

# neuropsych-pdf-parser

## Overview

The `neuropsych-pdf-parser` is the **first stage of the Luria neuropsych ingestion pipeline**. Its sole responsibility is document ingestion: turning a raw PDF (or URL) into a clean, structured, PHI-free text representation ready for downstream processing. It explicitly does *not* interpret, score, or summarize content — that is delegated to later pipeline stages.

## Role in Pipeline

This agent is positioned as Stage 1 in a multi-stage [[concepts/luria-neuropsych-pipeline]]. Its output feeds directly into a data extractor (Stage 2), which performs semantic interpretation and scoring analysis.

## Input

- **Local PDF path** (preferred): uses `Read` for short documents, `PageIndex` tools for PDFs longer than 20 pages.
- **URL**: uses `WebFetch` for HTML or downloads before processing.

## Workflow Steps

1. **Source identification** — determines local vs. remote and selects the appropriate tool.
2. **Long-document handling** — for PDFs >20 pages, calls `process_document` → `get_document_structure` → `get_page_content` on targeted page ranges. Avoids blind full-document reads.
3. **Document classification** — assigns exactly one of five types:
   - `neuropsych_assessment_report` (WAIS, WIAT, WMS, RBANS, MoCA, MMSE, CPT, BRIEF, etc.)
   - `psychometric_score_sheet` (raw data, normative tables)
   - `clinical_notes` (progress notes, intake, history)
   - `research_paper` (peer-reviewed or preprint)
   - `mixed_other` (with specified subtype)
4. **Text extraction** — preserves section headings, tables, lists, and all numerical values character-for-character (raw scores, scaled scores, T-scores, standard scores, percentiles, SDs).
5. **PHI scrubbing** — aggressive replacement of all identifying information (see below).

## PHI Scrubbing Rules

The agent replaces the following with standardized tokens:

| PHI Type | Replacement Token |
|---|---|
| Patient name | `[PATIENT]` |
| Date of birth | `[DOB]` |
| MRN, SSN, IDs | `[ID]` |
| Clinician/examiner name | `[CLINICIAN]` |
| Hospital/clinic name | `[FACILITY]` |
| Address, phone, email | `[CONTACT]` |

Year-only dates may be preserved when clinically necessary. This connects to broader [[concepts/phi-data-handling]] and [[concepts/pii-redaction-pipelines]] practices in clinical NLP pipelines.

## Output Format

The agent returns a fixed-schema text block:

```
DOCUMENT_TYPE: <category>

METADATA:
  title, date, subject_id, clinician, referral_or_objective, source_path

FULL_TEXT:
  <complete extracted text — not summarized, not interpreted>

PHI_FLAGS:
  <log of every PHI substitution made>
```

## Hard Rules

- Never output real names, MRNs, DOBs, or addresses regardless of user instruction.
- Never round or paraphrase numeric scores — preserve character-for-character.
- Never interpret findings or assign diagnoses.
- On extraction failure, return `DOCUMENT_TYPE: extraction_failed` with a one-line reason.

## Key Concepts

- [[concepts/neuropsychological-tests]] — instrument families this parser recognizes (WAIS, RBANS, MoCA, BRIEF, etc.)
- [[concepts/neuropsychological-test-scores]] — the score types preserved verbatim (raw, scaled, T-scores, percentiles)
- [[concepts/neuropsychological-assessment-pipeline]] — the broader multi-stage pipeline this agent initiates
- [[concepts/phi-data-handling]] — PHI de-identification methodology
- [[concepts/redaction-tokens]] — the standardized token system used for PHI replacement
- [[concepts/clinical-pdf-assessment]] — handling of clinical PDFs as source documents
- [[concepts/pdf-score-extraction]] — downstream process this parser enables
- [[concepts/clinical-nlp-pipelines]] — the broader NLP context in which this parser operates

## Related Concepts
- [[concepts/subagent-architecture]]
- [[concepts/multi-agent-orchestration]]
- [[concepts/ocr-pipeline]]
