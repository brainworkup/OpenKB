---
doc_type: short
full_text: sources/responses_to_claude.md
---

# Responses to Claude

## Overview
This document captures the user's replies to Claude's recommendations regarding plugins and subagent architecture for their neuropsychology data pipeline, clarifying current design decisions and recent structural changes.

## Plugin Recommendation: docx/xlsx/pdf Bundle
Claude suggested adopting the `anthropic-agent-skills` docx/xlsx/pdf bundle for ingesting PDFs, indexing spreadsheets, and producing reports via Quarto.

### User Clarification
- Google Sheets integration was **removed** approximately one month ago.
- The system now runs exclusively through the **`engine/cingulate`** engine (formerly `R/cingulate`).
- Data storage formats: **CSV, Arrow, Parquet** — not spreadsheets. See [[concepts/parquet-as-knowledge-store]].
- Report generation uses a [[concepts/quarto]] / [[concepts/typst-typesetting]] pipeline.
- The user renamed/moved the folder structure to `engine/cingulate` under `~/luria` and expressed uncertainty about whether this was a good decision.
- A recent reorganization introduced bugs: old code re-appeared and structural inconsistencies emerged.

## Subagent Recommendation: Promote to `.claude/agents/`
Claude suggested mirroring `subagents/*` specs into `.claude/agents/<name>.md` with frontmatter so the Task tool could invoke them in parallel.

### Relevant Subagents Mentioned
- `subagents/Neuropsych_Data_Extractor/`
- `subagents/PDF_Ingestion_Parser/`
- `subagents/Narrative_Report_Generator/`
- `subagents/Sheets_Data_Indexer/`

These are currently documented as `AGENTS.md + tools.json` specs (per memory #1140), not dispatchable subagents.

## Key Themes
- [[concepts/cingulate-engine]] — The core data processing engine replacing earlier Google Sheets workflows.
- [[concepts/subagent-architecture]] — Design of parallel, dispatchable agents for intake, extraction, and narrative generation.
- [[concepts/narrative-report-generation]] — Report generation pipeline used in place of spreadsheet outputs.
- [[concepts/clinical-data-management]] — Preference for CSV/Arrow/Parquet over XLSX/Sheets.
- [[concepts/neuropsychological-assessment-pipeline]] — Overall intake → extraction → narrative pass the subagents support.
- Ongoing tension between **documentation-as-spec** (see [[concepts/documentation-as-code]]) vs. **dispatchable agent** patterns (see [[concepts/multi-agent-orchestration]]).
- Risk of **structural drift** after reorganizations — old code resurfacing is a noted hazard.

## Open Questions
- Was renaming `R/cingulate` → `engine/cingulate` under `~/luria` the right structural choice?
- How should `Sheets_Data_Indexer` be repurposed or retired given the removal of Google Sheets?
- Should subagent specs be promoted to `.claude/agents/` to enable parallel Task invocation?

## Related Concepts
- [[concepts/pdf-score-extraction]]
