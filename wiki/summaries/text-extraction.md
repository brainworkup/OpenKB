---
doc_type: short
full_text: sources/text-extraction.md
---

# Text Extraction Component

## Overview

The text extraction component handles ingestion of historical neuropsychological reports from multiple file formats into plain text, preparing them for downstream [[concepts/neuropsychological-assessment-pipeline]] processing. Implemented in `soul/neuro_report_style_agent.py` (lines 54–67).

## Core Function: `extract_text(path: Path) -> str`

### Supported Formats

| Format | Extension | Implementation |
|--------|-----------|----------------|
| Plain Text | `.txt`, `.md` | Native Python `path.read_text()` |
| PDF | `.pdf` | PyPDF2 `PdfReader` |

### Error Handling

- **Unsupported file type**: Raises `ValueError` — `f"Unsupported file type: {path}"`
- **Missing PyPDF2**: Raises `RuntimeError` with installation instructions
- **Encoding issues**: Falls back to UTF-8 with `errors="ignore"`

### File Discovery

`iter_report_files()` is a generator that recursively finds all report files matching `*.pdf`, `*.txt`, and `*.md` patterns using `rglob`.

## Chunking Strategy

After extraction, text is passed to `chunk_text()` for [[concepts/text-chunking]] before embedding:

```python
chunks = chunk_text(text, chunk_size=1200, overlap=150)
```

### Parameters & Rationale

- **`chunk_size=1200`**: Balances context preservation with embedding quality
- **`overlap=150`**: Maintains continuity across chunk boundaries
- **Whitespace normalization**: Collapses multiple spaces/newlines before chunking

## Pipeline Integration

The extraction component sits at the start of the ingestion pipeline, feeding into [[concepts/sqlite-as-vector-store]] for storage:

```
Report Files → extract_text() → chunk_text() → embed_with_fallback() → SQLite
```

See [[concepts/retrieval-augmented-generation]] for downstream processing details.

## Dependencies

- `PyPDF2` — optional, required only for PDF support
- Standard library `pathlib.Path`

## Testing Recommendations

- Multi-page PDFs
- UTF-8 encoded text files
- Markdown with formatting
- Files with edge whitespace

## Related Concepts
- [[concepts/clinical-nlp-pipelines]]
- [[concepts/pdf-score-extraction]]
- [[concepts/ocr-pipeline]]
- [[concepts/vector-search]]
- [[concepts/local-first-architecture]]
- [[concepts/clinical-data-privacy]]
- [[concepts/python-project-structure]]
