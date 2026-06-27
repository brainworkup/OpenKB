---
doc_type: short
full_text: sources/style-training-to-report-drafting.md
---

# Style Training → Report Drafting

## Overview

This document describes a multi-step workflow that uses a **Soul style agent** (`neuro_report_style_agent.py`) to learn a clinician's writing style from historical neuropsychological reports and apply it to draft new report sections. The output integrates into a [[concepts/quarto]] rendering pipeline for final PDF production.

## Pipeline Architecture

The workflow proceeds in four major stages:

1. **Build Embedding Index** — Historical reports (PDF/TXT/MD) are chunked and embedded into a [[concepts/sqlite-as-vector-store]] (`report_style_index.sqlite`) using a local embedding model.
2. **Train Style Profile** — The index is queried to extract a `report_style_profile.json` capturing voice, tone, structure patterns, domain phrases, and writing rules (`do_rules`, `avoid_rules`, `typical_phrases`).
3. **Draft Report Section** — A `write-report` command retrieves top-k similar chunks (cosine similarity) and injects the style profile as a system prompt to generate a prose draft.
4. **Quarto Rendering** — Draft text is manually copied into `.qmd` section files and rendered to PDF via `quarto render`.

## Key Components

- **`neuro_report_style_agent.py`** — CLI agent with subcommands: `build-index`, `train-style`, `write-report`
- **OMLX Server** — Local inference server at `http://127.0.0.1:8000/v1`; supports model overrides (e.g., Ollama). See [[concepts/omlx-server]].
  - Embedding model: `nomicai-modernbert-embed-base-bf16`
  - Generation model: `Qwopus3.5-9B-v3-PolarQuant-MLX-4bit`
- **`report_style_profile.json`** — Serialized style profile injected into the system prompt at generation time. See [[concepts/style-profile-extraction]].
- **`report_style_index.sqlite`** — [[concepts/sqlite-as-vector-store]] storing embedded report chunks for retrieval
- **`.qmd` section files** — Quarto Markdown partials included via `{{< include >}}` in `template.qmd`. See [[concepts/modular-report-architecture]].

## Integration Points

| From | To | Mechanism |
|---|---|---|
| `report_style_profile.json` | `write-report` | Injected as JSON system prompt |
| `report_style_index.sqlite` | `write-report` | Top-k cosine-similarity retrieval |
| `draft_report.txt` | `.qmd` sections | Manual copy-paste |
| `.qmd` sections | PDF | Quarto `{{< include >}}` + render |

## Configuration & Overrides

All CLI flags are overridable, allowing substitution of:
- Alternative inference backends (Ollama)
- Different embedding or generation models
- Retrieval depth (`--top-k`)
- Generation temperature (`--temperature`)

## Quality Checklist

- Style profile `voice` matches clinician's narrative style
- `do_rules` enforce plain language and active voice
- `avoid_rules` prevent passive voice and jargon
- `typical_phrases` include domain terminology (e.g., "intelligent testing")
- Clinician reviews draft before `.qmd` insertion
- `[NEEDS DATA]` placeholders replaced with real patient data

## Related Concepts
- [[concepts/neuropsychological-reporting]]
- [[concepts/clinical-report-structure]]
- [[concepts/local-first-architecture]]
- [[concepts/clinical-data-privacy]]

- [[concepts/retrieval-augmented-generation]] — The retrieval mechanism (top-k cosine similarity chunks as context)
- [[concepts/style-profile-extraction]] — Learning voice/tone from historical documents
- [[concepts/quarto]] — The rendering system consuming drafted sections
- [[concepts/sqlite-as-vector-store]] — Embedded chunk storage and retrieval
- [[concepts/local-llm-inference]] — OMLX/Ollama serving models locally
- [[concepts/narrative-report-generation]] — The broader goal of AI-assisted clinical report drafting
- [[concepts/text-chunking]] — Underlying mechanism for splitting historical reports before embedding
- [[concepts/vector-search]] — Cosine-similarity retrieval used to prepend context at generation time
- [[concepts/single-file-agent-pattern]] — Architectural pattern followed by `neuro_report_style_agent.py`