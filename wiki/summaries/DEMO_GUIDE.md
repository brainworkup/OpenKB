---
doc_type: short
full_text: sources/DEMO_GUIDE.md
---

# Summary: Luria Neuropsych Pipeline — YC Demo Guide

**Source**: DEMO_GUIDE
**Status**: Ready for Demo (2025-01-21)
**Demo URL**: http://127.0.0.1:8501

## Overview

Luria is a clinical neuropsychology report processing system that transforms raw PDF reports into structured data and generates new clinical-quality reports matching a clinician's writing style. This guide covers quick-start setup, a recommended 5-minute demo script, architecture overview, and troubleshooting for a YC investor demo.

## Quick Start

```bash
cd /Users/joey/luria/app/streamlit
uv sync
uv run streamlit run app.py --server.address 127.0.0.1
```

## Core Features Demonstrated

### 1. PDF Upload & Clinical Data Extraction
- Drag-and-drop neuropsychological PDF reports into the "Ingest" tab
- Runs a **4-stage [[concepts/clinical-nlp-pipelines]]**:
  1. **Parse**: Docling extracts text + local PHI redaction (no network, see [[concepts/privacy-first-software]])
  2. **Extract**: Claude (Anthropic API) converts narrative to structured JSON
  3. **Index**: SQLite + LanceDB vector store
  4. **Report**: oMLX local LLM generates narrative clinical report

### 2. Summary & Findings Generation
- Auto-generates an Executive Summary after ingestion
- Groups scores by cognitive domain
- Applies normative benchmarks (T-scores, percentiles)
- Highlights significant discrepancies (>1.5 SD)
- Generates evidence-based recommendations
- Downloadable as Markdown report

### 3. "Luria Voice" — Style Matching via RAG
- Sidebar toggle: "Inject SOUL (style profile + exemplars)"
- Uses [[concepts/retrieval-augmented-generation]] to retrieve similar historical report snippets via semantic search
- Injects style guidance into LLM prompt to match tone/phrasing without copying patient facts
- Configured via `NEUROPSYCH_SOUL_DB` and `NEUROPSYCH_SOUL_PROFILE` environment variables
- Key file: `neuropsych_agent/tools/soul_context.py`

### 4. Export of Polished Report Artifacts
- Multiple output formats:
  - **Markdown**: Always available, professional quality
  - **Typst** (`.typ` + compiled PDF): Print-ready, available via `brew install typst`
  - **Quarto PDF**: Publication-quality (requires R + Quarto)
  - **DOCX**: Editable (optional)
- Quarto styles: `neurotyp-adult-typst`, `neurotyp-pediatric-typst`, `neurotyp-forensic-typst`

## Recommended 5-Minute Demo Script

| Time | Action |
|------|--------|
| 0:00–0:30 | Introduction: explain Luria's purpose |
| 0:30–2:00 | PDF ingestion — drag-drop, run pipeline, narrate 4 stages |
| 2:00–3:00 | Show results — Executive Summary + download buttons |
| 4:00–5:00 | RAG Query in "Ask" tab (e.g., "What were the memory scores?") |
| Bonus | Voice/SOUL demo if time permits |

## Architecture

```
Streamlit UI → LangGraph Orchestration
    ├── parse_node  (Docling)
    ├── extract_node (Claude API)
    ├── index_node   (SQLite + LanceDB)
    └── report_node  (Claude API + Typst/Quarto)
→ Storage: SQLite | LanceDB | File system
```

## Key Environment Variables

- `ANTHROPIC_API_KEY` — Claude extraction (recommended)
- `OMLX_BASE_URL` / `OMLX_CHAT_MODEL` — Local LLM (fully offline)
- `VOICE_ROOT`, `NEUROPSYCH_SOUL_DB`, `NEUROPSYCH_SOUL_PROFILE` — Voice/style features

## Key Files Reference

| Component | Path |
|-----------|------|
| Streamlit App | `app/streamlit/app.py` |
| Pipeline Graph | `neuropsych_agent/graph.py` |
| Pipeline Nodes | `neuropsych_agent/nodes.py` |
| PDF Parser | `tools/pdf_parser.py` |
| Soul Context | `tools/soul_context.py` |
| Extractor Prompt | `subagents/Neuropsych_Data_Extractor/AGENTS.md` |
| Generator Prompt | `subagents/Narrative_Report_Generator/AGENTS.md` |

## Troubleshooting Highlights

- Missing `ANTHROPIC_API_KEY`: UI still loads; can browse existing data
- `extract_node` failure: Check `subagents/Neuropsych_Data_Extractor/AGENTS.md` existence
- No Typst PDF: `brew install typst`
- No Quarto PDF: Install R + Quarto (optional — Markdown + Typst are sufficient)

## Verification Commands

```bash
python3 scripts/smoke_test_paths.py
python3 demo_readiness_check.py
```

## Related Concepts
- [[concepts/persistent-memory]]

- [[concepts/clinical-nlp-pipelines]] — LangGraph StateGraph orchestration for clinical NLP
- [[concepts/privacy-first-software]] — Local PHI redaction in clinical contexts
- [[concepts/retrieval-augmented-generation]] — RAG for voice/style matching