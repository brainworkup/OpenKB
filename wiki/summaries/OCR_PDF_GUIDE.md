---
doc_type: short
full_text: sources/OCR_PDF_GUIDE.md
---

# OCR PDF Processing Guide for RAG System

**Source:** OCR_PDF_GUIDE
**Created:** January 16, 2026
**Purpose:** Convert scanned/OCR PDFs into RAG-compatible formats

## Overview

Scanned PDFs often lack usable text layers, contain garbled OCR output, or store text as images — all of which degrade [[concepts/retrieval-augmented-generation]] embedding and retrieval quality. This guide provides four OCR solutions with R integration code, plus a recommended end-to-end workflow.

## Core Problem

OCR PDFs present these challenges for [[concepts/retrieval-augmented-generation]] ingestion:
- No or poor text layer
- Garbled searchable text
- Images of text rather than actual text
- Formatting issues confusing parsers

## OCR Solutions (Ranked)

### 1. OCRmyPDF ⭐ (Recommended for Batch)
- **Engine:** Tesseract 5.0+
- **Strength:** Best quality-to-speed ratio; preserves PDF format; handles existing text layers with `--skip-text`
- **Install:** `brew install ocrmypdf` or `conda install -c conda-forge ocrmypdf`
- **Key flags:** `--deskew`, `--clean`, `--rotate-pages`, `--optimize 3`, `--skip-text`, `--force-ocr`
- **R integration:** `system2("ocrmypdf", args = c(...))` for single files or batch loops
- **Parallelism:** `find . -name "*.pdf" | parallel -j 4 ocrmypdf {} ocr_{}`

### 2. Adobe Acrobat Pro ⭐ (Best Quality)
- **Strength:** Professional-grade OCR with excellent formatting preservation
- **Workflow:** Tools → Scan & OCR → Recognize Text; batch via Action Wizard
- **Settings:** English, Searchable Image output, 300 DPI
- **Cost:** Requires paid subscription

### 3. Tesseract (Open Source, Flexible)
- **Packages:** `tesseract` + `pdftools` in R
- **Workflow:** Convert PDF pages to images at 300 DPI → OCR each page → combine with `--- PAGE BREAK ---` separators → save as `.md`
- **Output format:** Markdown text files (not PDFs)

### 4. Google Cloud Vision API (Complex Layouts)
- **Strength:** AI-powered, superior for multi-column documents, tables, handwriting
- **Cost:** 1,000 pages/month free, then $1.50/1,000 pages
- **R package:** `googleCloudVisionR`

## Recommended Workflow (4 Steps)

### Step 1: Assess PDFs
```r
check_pdf_text(pdf_path)  # Returns: NO_TEXT / POOR_TEXT / GOOD_TEXT
assess_pdfs(folder_path)  # Returns tibble of all PDFs and status
```
Threshold: <100 chars = needs OCR; <1000 chars = may need OCR.

### Step 2: Choose Method
- Bulk → OCRmyPDF
- Selective → Tesseract in R

### Step 3: Process with Pipeline
`prepare_ocr_pdfs_for_rag()` function handles:
- Output folder creation
- Per-file tryCatch error handling
- Progress reporting via `cli`
- Returns paths of successfully processed files

### Step 4: Rebuild RAG
Re-run `setup_shared_rag()` after OCR-processed files are in place.

## Quality Settings

| DPI | Use Case |
|-----|----------|
| 300 | Minimum acceptable |
| 400 | Recommended for small text |
| 600 | Maximum (diminishing returns) |

**Pre-processing steps:** deskew, clean (artifact removal), rotate, crop.

**Validation:** Compare `sum(nchar(pdf_text(...)))` before and after OCR.

## Utility Script: `ocr_utils.R`

Complete reusable script providing:
- `ocr_single_pdf(input_path, output_path, overwrite)` — single file OCR with existence check
- `ocr_folder(folder_path, suffix, in_place)` — batch processing, skips already-OCR'd files, supports in-place replacement

Save to: `/Users/joey/reports/_shared_references/ocr_utils.R`

## Key Concepts
- [[concepts/ocr-pipeline]] — the OCR processing strategies and tools described throughout
- [[concepts/retrieval-augmented-generation]] — the RAG system this pipeline feeds
- [[concepts/clinical-pdf-assessment]] — clinical PDF documents (e.g., neuropsych reports) as a primary use case
- [[concepts/pdf-score-extraction]] — downstream use of OCR-cleaned PDFs for structured data extraction
- [[concepts/neuropsychological-assessment-pipeline]] — broader context for why clean PDF text matters
- [[concepts/r-python-integration]] — R-based automation patterns used throughout

## Related Concepts
- [[concepts/parquet-as-knowledge-store]]
- [[concepts/privacy-first-software]]
- [[concepts/vector-search]]
