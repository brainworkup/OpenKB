---
doc_type: short
full_text: sources/style-trainer.md
---

# Style Trainer Component

## Overview

The Style Trainer is a module within `soul/neuro_report_style_agent.py` (lines 238–279) that analyzes indexed historical neuropsychological reports to extract a reusable writing style profile. The profile captures voice, tone, structural patterns, and phrasing conventions for use in downstream report generation. See also [[concepts/style-profile-extraction]] and [[summaries/soul-style-agent]].

## Core Workflow

1. **Query DB** — Retrieves style-exemplar chunks from a SQLite index using a fixed [[concepts/semantic-search|semantic seed query]].
2. **Corpus Assembly** — Formats top-k chunks with source attribution (`[Source: path#chunk_id]`).
3. **LLM Analysis** — Sends corpus to an LLM at low temperature (0.1) for consistent extraction.
4. **Profile Save** — Parses and writes a structured JSON profile to disk.

## Exemplar Retrieval

- Uses a fixed seed query focused on writing style, narrative voice, phrasing, section structure, and professional tone.
- Embeds the query via `embed_with_fallback` and runs similarity search via `retrieve_top_k`.
- Default: **12 exemplar chunks** (`--style-examples 12`).
- Relies on [[concepts/sqlite-as-vector-store]] for indexed chunk storage and retrieval.

## Style Profile Schema

The output JSON profile contains six keys:

| Key | Type | Description |
|---|---|---|
| `voice` | string | Overall narrative voice (e.g., professional clinical) |
| `tone` | string | Emotional register (e.g., objective yet empathetic) |
| `structure_patterns` | array | Ordering conventions (e.g., demographics → findings → recommendations) |
| `typical_phrases` | array | Characteristic sentence starters and frames |
| `do_rules` | array | Positive style rules (e.g., person-first language, include percentiles) |
| `avoid_rules` | array | Negative style rules (e.g., no casual contractions, no speculation beyond data) |

See [[summaries/0004-soul-style-profile-json]] for the profile format in broader context.

## CLI Usage

```bash
python neuro_report_style_agent.py train-style \
    --db-path ./report_style_index.sqlite \
    --profile-path ./report_style_profile.json \
    --style-examples 12
```

## Error Handling

- Raises `RuntimeError` if no indexed chunks are found (requires `build-index` to run first).
- `json.JSONDecodeError` propagates if the LLM returns malformed JSON.
- Missing DB file causes a SQLite connection error.

## Iteration and Versioning

- Style profiles are **not auto-updated**; regeneration requires re-running `build-index` then `train-style`.
- Profile versioning (`v1.json`, `v2.json`) is recommended to support A/B testing across clinician styles or time periods.
- Feeds into [[concepts/narrative-report-generation]] pipelines that consume the profile for consistent output.

## Key Design Decisions

- **Low temperature (0.1)**: Minimizes variance in style extraction, ensuring reproducibility.
- **Fixed seed query**: Guarantees consistent exemplar retrieval across runs regardless of corpus changes.
- **Source attribution in corpus**: Preserves traceability of which documents influenced the profile.
- Integrates with the broader [[concepts/neuropsychological-assessment-pipeline]] and relies on [[concepts/llm-provider-abstraction]] via the `generate_with_fallback` and `embed_with_fallback` helper functions.
- Related to [[concepts/single-file-agent-pattern]] as described in [[summaries/0008-soul-single-file-style-agent-architecture]].

## Related Concepts
- [[concepts/retrieval-augmented-generation]]
- [[concepts/vector-search]]
- [[concepts/semantic-search]]
- [[concepts/fallback-strategy]]
- [[concepts/clinical-report-structure]]
- [[concepts/neuropsychological-reporting]]
