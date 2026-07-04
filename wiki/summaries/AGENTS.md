---
doc_type: short
full_text: sources/AGENTS.md
---

# AGENTS Project Overview

The AGENTS project is a local neuropsychological report soul agent powered by a local LLM + lightweight RAG. The pipeline has three stages: building an index from PDF/TXT/MD reports, training a soul profile from indexed chunks, and drafting a new report section.

## Architecture

The pipeline consists of three CLI subcommands:
1. **`build-index`** --- Extracts text from `neuropsych_report_pdf_bucket/`, chunks with overlap, embeds each chunk via OMLX, stores everything in a SQLite DB (`report_soul_index.sqlite`).
2. **`train-soul`** --- Runs a seed RAG query over the index to retrieve soul-exemplar chunks, then prompts the LLM to return a compact JSON soul profile saved to `report_soul_profile.json`. 
3. **`write-report`** --- Embeds the user prompt, retrieves top-k relevant chunks, assembles a prompt with the soul profile + retrieval context, and calls the LLM to produce a draft.

## Local Inference (OMLX)

The script calls an **OMLX** server (OpenAI-compatible API) running locally at `http://127.0.0.1:8000/v1`. The helper functions `embed_with_fallback` and `generate_with_fallback` are the single call-site for all LLM I/O.

## Important: Bash Sandboxing Issue

**Do NOT run `python main.py build-index` from bash** (e.g., via `uv run` or directly in terminal). Bash is sandboxed in Positron and cannot reach localhost:8000. 

Instead, run from the Python console:
```python
import sys; sys.path.insert(0, '/Users/joey/luria/voice/soul')
from cli import build_arg_parser
from core.database import build_index
parser = build_arg_parser()
args = parser.parse_args(['build-index', '--reports-dir', './reports/neuropsych_report_pdf_bucket', '--db-path', './soul_db/report_soul_index_sections.sqlite', '--aliases-path', './section_aliases.json'])
build_index(args)
```

## Related Concepts
- [[concepts/agents-local-inference-reliability]]
- [[concepts/agents-single-file-agent-pattern]]
- [[concepts/concepts-attention]]
- [[concepts/explorations-neuropsychological-test-scores]]
