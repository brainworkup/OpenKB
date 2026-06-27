---
doc_type: short
full_text: sources/customization.md
---

# Customization Workflows

## Overview

This document provides comprehensive guidance for tailoring the neuro report style agent (see [[summaries/soul-style-agent]]) to specific clinicians, patient populations, and output requirements. It covers profile management, chunking strategies, retrieval tuning, model configuration, section templates, output formats, performance optimization, and quality assurance.

## Custom Style Profiles

### Per-Clinician Profiles
Individual clinicians can have dedicated [[concepts/style-profiles]] trained exclusively from their own historical reports. The workflow involves:
1. Building an index from a clinician-specific reports directory.
2. Training a profile saved as a `.json` file.
3. Passing that profile at report-generation time.

### Per-Population Profiles
Separate profiles can be maintained for distinct populations:
- **Pediatric** — developmental focus, age-appropriate language.
- **Adult** — standard neuropsychological framing.
- **Forensic** — heightened objectivity and precision requirements (see [[concepts/forensic-neuropsychological-evaluation]]).

### Manual Profile Editing
JSON profiles support fine-grained manual editing. Key configurable fields:
- `voice` — narrative persona (e.g., "Professional clinical analyst with developmental focus").
- `tone` — affective register (e.g., "Objective yet empathetic, strength-based").
- `structure_patterns` — ordered list of structural conventions.
- `do_rules` — enforced inclusions (e.g., person-first language, percentile ranks).
- `avoid_rules` — enforced exclusions (e.g., definitive diagnostic statements, contractions).

## Custom Chunking Strategies

Chunk size affects how reports are indexed and retrieved (see [[concepts/rag-chunking]] and [[concepts/text-chunking]]):

| Use Case | `--chunk-size` | `--overlap` |
|---|---|---|
| Long-form comprehensive reports | 2000 | 200 |
| Brief screening reports | 600 | 75 |
| Default | ~1000 | ~100 |

Section-aware chunking can be achieved by pre-extracting named sections before indexing, attaching section metadata for more targeted retrieval.

## Custom Retrieval Parameters

Retrieval behavior is tuned via `--top-k` (number of retrieved chunks) and `--temperature` (generation randomness):

| Mode | `--top-k` | `--temperature` |
|---|---|---|
| High-context (complex cases) | 12 | 0.15 |
| Quick drafting (standard cases) | 4 | 0.25 |
| Balanced (default) | 6 | 0.2 |

## Custom Model Configuration

- **Embedding model**: Override with `--omlx-embed-model` (e.g., `nomic-embed-text-v1.5`). See [[concepts/omlx-server]].
- **Generation model**: Override with `--omlx-gen-model` (e.g., `llama-3.1-70b`). See [[concepts/local-llm-inference]].
- **Alternative backends**: Ollama can be substituted by setting `OMLX_URL` to the Ollama endpoint (`http://localhost:11434/v1`), leveraging the [[concepts/openai-compatible-api]] interface.

## Custom Report Sections

### Template-Based Generation
Prompt engineering enables generation of targeted sections relevant to [[concepts/clinical-report-structure]]:
- Executive summary (referral question, key findings, impressions, recommendations).
- Background/history (developmental, academic, prior evaluations).
- Test results as a markdown table (Test, Score, Percentile, Interpretation) drawing on [[concepts/neuropsychological-test-scores]].
- Recommendations organized by domain (school accommodations, home strategies, follow-up).

### Multi-Section Workflow
Each section can be generated independently and concatenated into a full report via shell scripting, following a [[concepts/modular-report-architecture]] approach that allows modular review and editing.

## Custom Output Formats

- **Markdown**: Default `--output report.md`.
- **Structured JSON**: Programmatic extension to emit sections and metadata as a JSON object, useful for downstream processing or EHR integration. Related to [[concepts/narrative-report-generation]].

## Performance Optimization

- **Batch indexing by year** for corpora of 1000+ reports, followed by database merging with a custom `merge_indices.py` script. See [[concepts/sqlite-as-vector-store]] for the underlying storage mechanism.
- **Parallel PDF extraction** using GNU `parallel` to accelerate ingestion.

## Quality Assurance Workflow

### Automated Checks
Post-generation shell checks include:
- Scanning for unfilled `[NEEDS DATA]` markers.
- Word count validation.
- Detecting prohibited casual contractions (e.g., `don't`, `can't`).

These checks complement the broader [[concepts/report-review-qa]] process.

### A/B Profile Testing
Two [[concepts/style-profiles]] can be compared by generating parallel drafts and running `diff` to evaluate stylistic differences, supporting iterative profile refinement via [[concepts/style-profile-extraction]].

## Related Pages
- [[summaries/soul-style-agent]]
- [[summaries/style-trainer]]
- [[summaries/style-training-to-report-drafting]]
- [[concepts/style-profiles]]
- [[concepts/rag-chunking]]
- [[concepts/retrieval-augmented-generation]]
- [[concepts/sqlite-as-vector-store]]
- [[concepts/narrative-report-generation]]
- [[concepts/modular-report-architecture]]

## Related Concepts
- [[concepts/local-first-architecture]]
- [[concepts/single-file-agent-pattern]]
- [[concepts/neuropsychological-reporting]]
- [[concepts/clinical-communication-register]]
- [[concepts/validity-language]]
- [[concepts/neuropsychological-report-variables]]
