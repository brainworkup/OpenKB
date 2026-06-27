---
doc_type: short
full_text: sources/full-pipeline.md
---

# Full Pipeline Workflow

This document describes the end-to-end workflow for the `neuro_report_style_agent.py` tool: indexing a corpus of historical neuropsychological reports, extracting a writing style profile, and generating new report drafts using [[concepts/retrieval-augmented-generation]] and a local LLM.

## Prerequisites

- **OMLX local LLM server** running on `http://127.0.0.1:8000` with two models:
  - `nomicai-modernbert-embed-base-bf16` — for embeddings (see [[concepts/multimodal-embeddings]])
  - `Qwopus3.5-9B-v3-PolarQuant-MLX-4bit` — for text generation via [[concepts/omlx-server]]
- A corpus of historical reports in `./neuropsych_report_pdf_bucket/` (`.pdf`, `.txt`, `.md` supported)
- Python dependencies installed via `uv sync` or `pip install PyPDF2`

## Stage 1: Build Index

Extracts text from all reports and stores chunked embeddings in a SQLite database.

```bash
python soul/neuro_report_style_agent.py build-index \
    --reports-dir ./neuropsych_report_pdf_bucket \
    --db-path ./report_style_index.sqlite \
    --chunk-size 1200 \
    --overlap 150
```

- Default chunk size: **1200 characters**, overlap: **150 characters**
- Output: SQLite database with a `chunks` table (see [[concepts/sqlite-as-vector-store]])
- Supports incremental updates via `INSERT OR REPLACE`
- Text chunking strategy described in [[concepts/text-chunking]] and [[concepts/rag-chunking]]

## Stage 2: Train Style

Analyzes indexed chunks to extract a writing style profile saved as JSON (see [[concepts/style-profile-extraction]] and [[concepts/style-profiles]]).

```bash
python soul/neuro_report_style_agent.py train-style \
    --db-path ./report_style_index.sqlite \
    --profile-path ./report_style_profile.json \
    --style-examples 12
```

The profile captures: `voice`, `tone`, `structure_patterns`, `typical_phrases`, `do_rules`, and `avoid_rules`.

## Stage 3: Write Report

Generates a new report draft by combining the style profile with RAG-retrieved context chunks, supporting [[concepts/narrative-report-generation]].

```bash
python soul/neuro_report_style_agent.py write-report \
    --db-path ./report_style_index.sqlite \
    --profile-path ./report_style_profile.json \
    --prompt "Write a concise summary for a 12-year-old with attention concerns." \
    --output ./draft_report.txt \
    --top-k 6 \
    --temperature 0.2
```

- `--top-k`: number of context chunks retrieved (default 6)
- `--temperature`: LLM sampling temperature (default 0.2)
- Output goes to a file or stdout

## One-Shot Pipeline Script

A bash script chains all three stages sequentially with `set -e` for fail-fast behavior. Configuration variables (`REPORTS_DIR`, `DB_PATH`, `PROFILE_PATH`, `OUTPUT_PATH`) are set at the top.

## Incremental Updates

- **New reports**: Re-run `build-index`; existing chunks are updated via `INSERT OR REPLACE`.
- **Iterative drafting**: Run `write-report` at multiple temperatures (e.g., 0.1, 0.2, 0.3) to compare conservative vs. creative drafts.

## Best Practices

- Minimum **20–30 reports** recommended for meaningful style extraction
- Always review the generated style profile before use
- **Never use drafts without clinician review** (see [[concepts/report-review-qa]])
- Use `[NEEDS DATA]` as a signal for LLM hallucinations
- Version style profiles for reproducibility

## Key Concepts

- [[concepts/retrieval-augmented-generation]] — context retrieval driving generation
- [[concepts/sqlite-as-vector-store]] — vector and chunk storage backend
- [[concepts/style-profile-extraction]] — extracting and encoding writing style from exemplars
- [[concepts/style-profiles]] — structured JSON representation of writing style
- [[concepts/local-llm-inference]] — self-hosted inference via [[concepts/omlx-server]]
- [[concepts/single-file-agent-pattern]] — architecture of `neuro_report_style_agent.py`
- [[concepts/neuropsychological-reporting]] — clinical reporting context for generated drafts

## Related Concepts
- [[concepts/neuropsychological-assessment-pipeline]]
- [[concepts/clinical-data-privacy]]
- [[concepts/local-first-architecture]]
- [[concepts/pdf-score-extraction]]
- [[concepts/ocr-pipeline]]
