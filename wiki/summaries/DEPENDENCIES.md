---
doc_type: short
full_text: sources/DEPENDENCIES.md
---

# Luria Dependency & Tool Reference

**Source**: DEPENDENCIES · Auto-generated April 28, 2026

Comprehensive catalog of every package, external tool, and R library used across the Luria workspace — covering CLI operation, the Streamlit/RAG application, development tooling, and the LLM pipeline.

---

## Python Core Dependencies

Required for `import luria` and basic CLI operation:

- **Numeric/stats**: `numpy ≥1.24`, `pandas ≥2.0`, `scipy ≥1.10`
- **Visualization**: `matplotlib ≥3.7`, `seaborn ≥0.12`
- **ML**: `scikit-learn ≥1.3` (PCA, classifiers for [[concepts/cognitive-domains]] clustering)
- **Validation**: `pydantic ≥2.0` (strict mode for `PipelineState`)
- **CLI**: `click ≥8.0`, `typer ≥0.9`, `rich ≥13.0`
- **Config**: `pyyaml ≥6.0` ([[concepts/yaml-configuration]]), `python-dotenv ≥1.0`, `tqdm ≥4.65`

---

## Application Dependencies (Streamlit + Agent Pipeline)

Required for the full ingestion/RAG app at `app/streamlit/`:

- **[[concepts/langgraph-agent-workflows]]**: `langgraph` (core orchestration — `build_ingest_graph()`, `build_rag_graph()`)
- **LangChain ecosystem**: `langchain ≥1.2.15`, `langchain-core`, `langchain-anthropic ≥1.4.1`, `langchain-huggingface ≥1.2.2`, `langchain-docling`, `langchain-milvus ≥0.3.3`
- **[[concepts/lancedb-vector-store]]**: `lancedb` (local default at `data/vectors/`), optional `langchain-milvus`
- **[[concepts/multimodal-embeddings]]**: `sentence-transformers` (default: `all-MiniLM-L6-v2`, 384-dim)
- **LLM client**: `openai` package used for **[[concepts/omlx-server]]** (not OpenAI cloud) via [[concepts/openai-compatible-api]]
- **UI**: `streamlit ≥1.28` (4-tab interface: Ingest / Ask / KB / Audio)
- **Experimental**: `deepagents ≥0.5.3`, `honcho-ai ≥2.1.1`, `haystack-ai ≥2.28`

---

## Development & Documentation

- **Testing**: `pytest ≥7.0`, `pytest-cov ≥4.0`
- **Code quality**: `black ≥23.0`, `ruff ≥0.1`, `mypy ≥1.0`, `pre-commit ≥3.0`
- **Notebooks**: `ipython ≥8.0`, `jupyter ≥1.0`, `jupyterlab ≥4.0`
- **Docs**: `sphinx ≥7.0`, `sphinx-rtd-theme`, `myst-parser ≥2.0`, `sphinx-autodoc-typehints ≥2.0`

---

## External CLI Tools

| Tool | Purpose | Notes |
|---|---|---|
| **uv** ≥0.4 | Python package manager | [[concepts/uv-workspace-layout]] |
| **Typst** ≥0.11 | PDF typesetting | [[concepts/typst-typesetting]] forensic template fallback |
| **Quarto** ≥1.5 | Report rendering | [[concepts/quarto]] `neurotyp-*-typst` formats |
| **MacWhisper** | Audio transcription | `mw` binary on PATH |
| **[[concepts/omlx-server]]** | Local LLM server | [[concepts/openai-compatible-api]] `/v1` endpoint |
| **R** ≥4.3 | Statistical computing | `R_HOME` env var |

---

## R Dependencies

- **Tidyverse**: `dplyr`, `tidyr`, `ggplot2`
- **Psychometrics**: `psych`, `lavaan` (SEM / confirmatory factor analysis)
- **Interop**: `reticulate` ([[concepts/r-python-integration]]), `jsonlite`, `DBI`, `RSQLite`
- **Reporting**: `knitr`, `rmarkdown`
- **Internal**: `cingulate` — custom neuropsych analysis package at `agent/cingulate/` ([[concepts/cingulate-engine]])

See [[concepts/r-neuropsych-packages]] for broader R dependency context.

---

## oMLX Model Configuration

| Model | Purpose | Dim |
|---|---|---|
| `medgemma-1.5-4b-it-bf16` | Chat/Extraction (default) | — |
| `nomicai-modernbert-embed-base-bf16` | Embeddings + Reranking | 768 |
| `sentence-transformers/all-MiniLM-L6-v2` | Fallback embeddings | 384 |

oMLX exposes an [[concepts/openai-compatible-api]] `/v1` endpoint; the app uses `openai.OpenAI(base_url=...)` to call it locally. See [[concepts/local-llm-inference]] and [[concepts/mlx-framework]] for related context.

---

## Cloud LLM Providers (Optional)

- **Anthropic** (`langchain-anthropic`, `ANTHROPIC_API_KEY`): Claude extraction — PHI redacted before any cloud call
- **OpenAI** (`OPENAI_API_KEY`): GPT fallback (optional)
- **LangSmith** (`LANGSMITH_API_KEY`): Tracing via `LANGSMISH_TRACING=true`

See [[concepts/llm-provider-abstraction]] and [[concepts/fallback-strategy]] for architectural context.

---

## Storage Backends

| Backend | Path | Purpose |
|---|---|---|
| SQLite | `data/neuropsych.db` | Structured test scores + summaries (`Documents`, `TestScores`, `ClinicalSummaries`) |
| LanceDB | `data/vectors/` | Semantic search vectors (`narrative_chunks`) |
| SQLite (Soul) | `report_soul_index.sqlite` | Style exemplar vectors |
| DuckDB | `kb/store/` | Analytics / RAG column-store |

See [[concepts/lancedb-vector-store]], [[concepts/sqlite-as-vector-store]], and [[concepts/duckdb-as-vector-store]] for architectural details.

---

## LangGraph Pipeline Architecture

Two `StateGraph` pipelines powering the [[concepts/neuropsychological-assessment-pipeline]]:

1. **IngestGraph**: `START → parse → extract → index → report → END`
   - `parse`: Docling PDF → text (local, deterministic) — see [[concepts/docling-pdf-parsing]]
   - `extract`: LLM → JSON records (oMLX or Claude)
   - `index`: SQLite + LanceDB writes
   - `report`: LLM → markdown + optional Typst/Quarto

2. **RAGGraph**: `START → rag → END`
   - Semantic search (LanceDB) + SQL (`TestScores`) → LLM synthesis with citations — see [[concepts/retrieval-augmented-generation]]

**State object**: `PipelineState(TypedDict)` carries messages, pdf_path, doc_id, parsed, records, indexed, report, user_query, rag_answer, plus Voice keys.

---

## Subagents

| Subagent | Node | LLM? |
|---|---|---|
| PDF_Ingestion_Parser | `parse_node` | No |
| Neuropsych_Data_Extractor | `extract_node` | Yes (JSON extraction) |
| Sheets_Data_Indexer | `index_node` | No |
| Narrative_Report_Generator | `report_node` | Yes (markdown) |

See [[concepts/subagent-architecture]] and [[concepts/multi-agent-orchestration]] for design context.

---

## Luria Voice Stack

- **BRAND** (`voice/brand/`): Logos, colors, `_brand.yml` (Quarto brand extension) — see [[concepts/brand-theming]]
- **SOUL** (`voice/soul/`): Style profile + exemplar RAG (`report_soul_profile.json` + SQLite index) — see [[concepts/style-profiles]]
- **STYLE** (`voice/style/`): Quarto Typst formats (`neurotyp-adult-typst`, etc.) — see [[concepts/quarto-extensions]]

Soul agent (`voice/soul/neuro_report_soul_agent.py`): standalone CLI ([[concepts/single-file-agent-pattern]]) for building the exemplar index from de-identified prior reports.

---

## Security & PHI

- PHI redaction at [[concepts/docling-pdf-parsing]] parse stage — identifiers stripped before any cloud call (`PHI_REDACTION_ENABLED=true`)
- Local-first storage: SQLite + LanceDB on local disk only — see [[concepts/local-first-architecture]]
- oMLX enables fully offline chat/embeddings (`LLM_PROVIDER=omlx`) — see [[concepts/phi-data-handling]]
- `.env` secrets gitignored — see [[concepts/pii-redaction-pipelines]]

## Related Concepts
- [[concepts/luria-neuropsych-pipeline]]
- [[concepts/neuropsychological-toolkit]]
- [[concepts/python-environment-management]]
- [[concepts/vector-search]]
- [[concepts/clinical-nlp-pipelines]]
