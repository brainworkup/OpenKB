---
sources: [summaries/extract_pdf.md, summaries/full-pipeline.md, summaries/text-extraction.md, summaries/README.md, summaries/neuropsych-pdf-parser.md, summaries/SHINY_APP_FIXED.md, summaries/OCR_PDF_GUIDE.md]
brief: OCR pipelines convert scanned or image-based PDFs into machine-readable text for downstream processing.
---

# OCR Pipeline for Document Processing

An **OCR (Optical Character Recognition) pipeline** converts scanned or image-based PDFs into clean, machine-readable text. In knowledge-intensive workflows — particularly [[concepts/retrieval-augmented-generation]] and [[concepts/neuropsychological-assessment-pipeline]] — OCR quality directly determines how well documents can be embedded, indexed, and retrieved.

## Why OCR Pipelines Matter

Many real-world documents arrive as scanned PDFs: assessment reports, archival records, faxed forms. These files may have:
- No text layer at all (pure image)
- A garbled or incomplete searchable text layer
- Text stored as rasterized images at low DPI
- Skewed, rotated, or artifact-ridden pages

Without a preprocessing OCR stage, downstream systems — including [[concepts/vector-search]] and [[concepts/retrieval-augmented-generation]] — receive degraded or empty input, severely reducing retrieval quality.

A common upstream trigger for an OCR pipeline is a bulk extraction request: processing all PDFs in a folder to pull numbers, dates, and key information into a structured format (e.g., a spreadsheet). This use case — described in [[summaries/extract_pdf]] — illustrates how OCR is not just about text recovery, but about enabling structured [[concepts/pdf-data-extraction]] at scale.

## Core OCR Tools

### OCRmyPDF (Recommended for Batch)
- Wraps Tesseract 5.0+ to add a clean text layer directly into a PDF
- Preserves the original PDF format, making output compatible with standard PDF readers and parsers
- Key flags: `--deskew`, `--clean`, `--rotate-pages`, `--optimize 3`, `--skip-text`, `--force-ocr`
- Supports parallel processing via GNU `parallel`
- Best quality-to-speed ratio for bulk document workflows

### Tesseract (Open Source, Flexible)
- The underlying engine used by OCRmyPDF
- Can be used directly in R via the `tesseract` package combined with `pdftools`
- Workflow: convert PDF pages to images (300 DPI) → OCR each page → concatenate with page-break markers → save as Markdown
- Output is plain text/Markdown rather than an enriched PDF

### Adobe Acrobat Pro
- Highest quality for individual important documents
- Excellent layout and formatting preservation
- Supports batch processing via Action Wizard
- Requires paid subscription

### Google Cloud Vision API
- AI-powered OCR superior for complex layouts: multi-column, tables, handwriting
- Free tier: 1,000 pages/month; $1.50 per 1,000 pages beyond that
- Integrates with R via `googleCloudVisionR`

## Pipeline Stages

1. **Assessment** — Scan all PDFs and classify by text quality (`NO_TEXT`, `POOR_TEXT`, `GOOD_TEXT`) using character-count heuristics via `pdftools::pdf_text()`
2. **Pre-processing** — Deskew, clean artifacts, auto-rotate pages before OCR
3. **OCR execution** — Apply chosen engine (OCRmyPDF for bulk; Tesseract for selective)
4. **Extraction** — Pull structured data (numbers, dates, key fields) from the OCR'd text into a target format such as a spreadsheet or database table
5. **Validation** — Compare character counts before and after; spot-check first pages; verify extracted values against expected schemas
6. **Ingestion** — Feed OCR-processed files into downstream RAG or embedding pipeline

## DPI Recommendations

| DPI | Use Case |
|-----|----------|
| 300 | Minimum acceptable quality |
| 400 | Recommended for small or dense text |
| 600 | Maximum useful (diminishing returns, larger files) |

## Folder-Based Batch Workflows

A frequent real-world pattern is processing an entire folder of PDFs in one pass — extracting numbers, dates, and key information into a spreadsheet. This mirrors the workflow described in [[summaries/extract_pdf]] and is well-supported by:
- OCRmyPDF with GNU `parallel` for OCR layer injection
- [[concepts/docling-pdf-parsing]] for structured content extraction post-OCR
- [[concepts/pdf-score-extraction]] pipelines that map extracted text to scored fields
- DuckDB or similar tools for aggregating results across many files into tabular output (see [[concepts/duckdb-data-staging]])

## R Integration Pattern

The canonical R utility pattern (from [[summaries/OCR_PDF_GUIDE]]) wraps `system2("ocrmypdf", ...)` calls in two reusable functions:
- `ocr_single_pdf(input_path, output_path, overwrite)` — processes one file, skips if output already exists
- `ocr_folder(folder_path, suffix, in_place)` — batch processes a directory, skips already-OCR'd files

Progress reporting uses the `cli` package; errors are caught with `tryCatch` to allow partial success on large batches.

## Relationship to Other Concepts

- **[[concepts/retrieval-augmented-generation]]** — OCR is a prerequisite data-quality step; poor OCR directly degrades retrieval performance
- **[[concepts/clinical-pdf-assessment]]** — Neuropsychological and clinical assessment PDFs are a primary use case; scanned reports from referral sources commonly require OCR
- **[[concepts/pdf-score-extraction]]** — Structured score extraction from assessment PDFs depends on clean OCR output as input
- **[[concepts/pdf-data-extraction]]** — The broader goal of pulling structured data (numbers, dates, key facts) from PDFs into usable formats
- **[[concepts/phi-data-handling]]** — OCR pipelines processing clinical documents must account for protected health information in the resulting text
- **[[concepts/pii-redaction-pipelines]]** — OCR output may require redaction before storage or embedding
- **[[concepts/neuropsychological-assessment-pipeline]]** — End-to-end pipeline that OCR feeds into for report generation
- **[[concepts/docling-pdf-parsing]]** — A complementary parsing approach that can operate on OCR-enriched PDFs
- **[[concepts/duckdb-data-staging]]** — Useful for aggregating extracted fields from many OCR'd PDFs into a single queryable table
- **[[concepts/plain-text-documentation]]** — Tesseract-based pipelines output Markdown, aligning with plain-text-first documentation strategies

## See Also
- [[summaries/OCR_PDF_GUIDE]] — Full guide with installation instructions, R code, and workflow recommendations
- [[summaries/extract_pdf]] — Prompt template for bulk PDF data extraction into spreadsheet format
- [[summaries/AS_PROCESSING_COMPLETE]] — Assessment processing workflow context
- [[summaries/neuropsych-pdf-parser]] — Neuropsychology-specific PDF parsing patterns
- [[summaries/text-extraction]] — General text extraction strategies
- [[summaries/full-pipeline]] — End-to-end pipeline context
- [[summaries/README]] — Project overview
- [[summaries/SHINY_APP_FIXED]] — Related application context