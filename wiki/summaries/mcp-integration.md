---
doc_type: short
full_text: sources/mcp-integration.md
---

# MCP Integration Workflow

## Overview

This document describes the **Model Context Protocol (MCP)** integration used in the Voice Style system to power AI-driven neuropsychological report generation. All LLM operations run locally via [[concepts/local-llm-inference]] to ensure patient data privacy.

## Architecture

The system follows a three-tier architecture:

- **Voice Style Application** — top-level orchestrator
- **MCP Server (Local)** — standardized AI interface layer
- **Ollama (LLM Backend)** — local inference engine via HTTP API on `localhost:11434`

## Configuration (`config.yml`)

Key parameters (managed via [[concepts/yaml-configuration]]):

| Parameter | Purpose |
|---|---|
| `pdf_path` | Raw psychological test PDF input |
| `tree_path` | Output path for structured JSON |
| `llm_base_url` | Ollama API endpoint |
| `llm_model` | Model for inference (e.g., `ollama/llama3.1`) |
| `lookup_table` | CSV with clinical terminology mappings |

## MCP Tools

### 1. PDF Extraction Tool
Extracts structured data from [[concepts/neuropsychological-tests]] PDFs (e.g., WISC-V, WAIS-IV). Parses test scores, subtests, percentiles, and demographic info into a JSON structure reflecting [[concepts/neuropsychological-test-scores]].

### 2. Clinical Interpretation Tool
Generates narrative clinical interpretations from structured test data. Analyzes score patterns, compares to normative data, identifies strengths/weaknesses, and produces markdown-formatted output with domain-specific summaries aligned with [[concepts/neuropsychological-reporting]].

### 3. Lookup Table Integration
Maps raw test scores to standardized clinical terminology via a CSV lookup table, ensuring consistent descriptions and clinical labels.

## Workflow Steps

1. **Initialize MCP Client** — connect to local Ollama instance
2. **Invoke PDF Extraction** — call `extract_pdf_data` tool
3. **Process Structured Data** — load JSON for downstream R/neuro2 processing
4. **Generate Interpretations** — call `generate_interpretation` with domain context

This workflow is part of the broader [[concepts/neuropsychological-assessment-pipeline]] and relies on [[concepts/model-context-protocol]] as its interface standard.

## Privacy & Security

All processing is **local only** — no data leaves the machine, with no external API calls. This approach embodies [[concepts/privacy-first-software]] principles. The MCP server is restricted to localhost access, and [[concepts/phi-data-handling]] guidelines apply: PHI sanitization and anonymized identifiers are recommended.

## Performance

- **Model options**: `llama3.1` (balanced), `llama3.1:70b` (high quality), `mistral` (fast)
- **Caching**: Skip re-extraction if output JSON already exists
- **Batch processing**: `ThreadPoolExecutor` for parallel PDF handling

## Error Handling

- `ConnectionError`: Ollama not running (`ollama serve`)
- `ModelError`: Model not downloaded (`ollama pull llama3.1`)
- Data validation: checks for presence and non-emptiness of `tests` key

## Testing

- Unit tests for PDF extraction output structure
- Integration tests for full extraction-to-interpretation workflow
- Manual CLI testing via `python soul/extract_pdf_data.py`

## Future Enhancements

- Additional tools: Symptom Checker, Recommendation Generator, Quality Checker
- Model fine-tuning on clinical/domain-specific data
- Multi-modal support: scanned documents, OCR integration

## Related Concepts
- [[concepts/clinical-data-privacy]]
- [[concepts/clinical-nlp-pipelines]]
- [[concepts/retrieval-augmented-generation]]
- [[concepts/pii-redaction-pipelines]]

- [[concepts/local-llm-inference]] — local LLM inference backend via Ollama
- [[concepts/model-context-protocol]] — the MCP specification and interface standard
- [[concepts/neuropsychological-tests]] — test types (WISC-V, WAIS-IV) and score structures
- [[concepts/neuropsychological-assessment-pipeline]] — the broader report generation workflow
- [[concepts/phi-data-handling]] — patient data privacy and sanitization
- [[concepts/privacy-first-software]] — design philosophy for local-only processing
- [[concepts/yaml-configuration]] — configuration management via `config.yml`