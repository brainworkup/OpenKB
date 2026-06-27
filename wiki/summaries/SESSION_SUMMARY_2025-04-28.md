---
doc_type: short
full_text: sources/SESSION_SUMMARY_2025-04-28.md
---

# Luria Session Summary — 2025-04-28

## Overview

This session covered an architecture review and codemap update for the **Luria** neuropsychology platform following a significant refactor. The old CLI-based orchestration was replaced with a skill-based [[concepts/multi-agent-orchestration]] pattern, and the Streamlit app became the primary entry point.

## Architecture Changes

### Removed Components
- `app/cli.py` and `app/orchestrator.py` — CLI orchestration removed
- `app/agents/nse_agent.py`, `nt_agent.py` — Visit-scoped agents removed
- `app/intake/` — CSV extraction moved to the `cingulate` R package

### New / Active Components
- `app/streamlit/` — Main entry point
- `agent/cingulate/` — R6-based domain processing (see [[concepts/cingulate-engine]])
- `rag/page-index/` — Document processing service
- `rag/docling/` — PII obfuscation (renamed from `luria_docling/`)

Codemap saved to: `.omx/codemaps/luria-simplified-architecture-2025.md`

## Multi-Agent Orchestration Pattern

The new system uses a `luria-neuropsych-orchestrator` skill that delegates to sub-skills:

```
luria-neuropsych-orchestrator
├── luria-case-intake          → Patient data normalization
├── luria-score-processing     → Test score extraction
├── luria-interpretation       → Domain-level analysis
├── luria-report-writing       → Report generation
└── luria-quality-review       → Final validation
```

Each skill delegates to one or more agent layers:
- **Python [[concepts/langgraph-agent-workflows]] nodes** — PDF parse, extract, index, report
- **R6 Domain Processors** — IQ, Memory, Attention, Language, etc. (see [[concepts/cingulate-engine]])
- **PageIndex service agents** — Document upload, chat, export (see [[concepts/retrieval-augmented-generation]])

See [[concepts/multi-agent-orchestration]] and [[concepts/luria-neuropsych-pipeline]] for broader context.

## Key File Locations

| Component | Path |
|-----------|------|
| Streamlit App | `app/streamlit/app.py` |
| LangGraph Nodes | `app/streamlit/neuropsych_agent/nodes.py` |
| Graph Wiring | `app/streamlit/neuropsych_agent/graph.py` |
| SQLite Store | `app/streamlit/neuropsych_agent/tools/store.py` |
| Soul Context | `app/streamlit/neuropsych_agent/tools/soul_context.py` |
| R Workflow | `agent/cingulate/R/WorkflowRunnerR6.R` |
| Domain Processors | `agent/cingulate/R/DomainProcessorR6.R` |
| PageIndex Service | `rag/page-index/app/service.py` |
| Docling PII | `rag/docling/detect_pii.py` |
| Voice Soul | `voice/soul/neuro_report_soul_agent.py` |

## Streamlit App: Launch & Management

### Start (foreground)
```bash
cd /Users/joey/luria/app/streamlit
uv run streamlit run app.py \
    --server.address 127.0.0.1 \
    --server.port 8501 \
    --browser.gatherUsageStats false
```

### Start (background)
```bash
nohup uv run streamlit run app.py \
    --server.address 127.0.0.1 \
    --server.port 8501 \
    --browser.gatherUsageStats false \
    --server.headless true \
    > /tmp/streamlit.log 2>&1 &
```

### Status & Stop
```bash
curl http://localhost:8501/_stcore/health   # Health check
lsof -i :8501                              # Check process
pkill -f "streamlit.*8501"                # Stop server
tail -f /tmp/streamlit.log                # View logs
```

### Access URLs
- Direct: `http://localhost:8501`
- Windsurf proxy: `http://127.0.0.1:57764`

## Performance Notes

- **First run** is slow: UV creates `.venv` and installs 175+ packages
- **Docling models** download ~500MB on first PDF parse
- **Watchdog** improves file-watching: `pip install watchdog`
- **Local LLM** (OMLX) may have model load time on startup (see [[concepts/local-llm-inference]])

## Interface

The Streamlit app has a **4-tab interface**: Ingest, Reference, Ask, Knowledge.

## Next Steps (from session)

1. Review `.omx/codemaps/luria-simplified-architecture-2025.md`
2. Understand the skill-based orchestration pattern
3. Evaluate whether to restore any CLI functionality
4. Test the 4-tab Streamlit interface

## Related Concepts
- [[concepts/neuropsychological-assessment-pipeline]]
- [[concepts/r-python-integration]]
- [[concepts/pdf-score-extraction]]
- [[concepts/persistent-memory]]
- [[concepts/clinical-data-privacy]]
- [[concepts/multi-agent-orchestration]]
- [[concepts/langgraph-agent-workflows]]
- [[concepts/luria-neuropsych-pipeline]]
- [[concepts/cingulate-engine]]
- [[concepts/pii-redaction-pipelines]]
- [[concepts/retrieval-augmented-generation]]
- [[concepts/subagent-architecture]]
- [[concepts/sqlite-as-vector-store]]