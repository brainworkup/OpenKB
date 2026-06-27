---
sources: [summaries/AGE_OVERRIDE_GUIDE.md, summaries/2026-02-11-this-session-is-being-continued-from-a-previous-co.md, summaries/extract_pdf.md, summaries/full-pipeline.md, summaries/text-extraction.md, summaries/README.md, summaries/SESSION_SUMMARY_2025-04-28.md, summaries/neuropsych-pdf-parser.md, summaries/neuropsych-data-extractor.md, summaries/responses_to_claude.md, summaries/processed_files.md, summaries/WORKFLOW_INSTRUCTIONS.md, summaries/SHINY_APP_FIXED.md, summaries/REBUILD_FINAL_STATUS.md, summaries/REBUILD_COMPLETE.md, summaries/README_WORKFLOW.md, summaries/README_PIPELINE.md, summaries/QUICK_REFERENCE.md, summaries/OCR_PDF_GUIDE.md, summaries/KNOWLEDGE_BASE_EXPLAINED.md, summaries/FIX_EXPLANATION.md]
brief: Programmatically retrieving structured numerical scores from clinical assessment PDFs for downstream analysis.
---

# PDF Score Extraction

PDF score extraction refers to the process of programmatically retrieving structured numerical scores — such as T-scores, scaled scores, or index scores — from clinical assessment report PDFs. It is a core challenge in [[concepts/neuropsychological-assessment-pipeline]] workflows, where scores must flow from report PDFs into databases, templates, or downstream analysis tools.

A common starting point is a prompt-driven workflow: given a folder of PDFs, extract all numbers, dates, and key information into a spreadsheet format. This deceptively simple task hides significant complexity — particularly in clinical contexts where score formats vary widely and privacy requirements are strict.

---

## The Core Challenge

Not all PDFs are created equal. A PDF may contain a rich text layer and still be impossible to parse for specific scores. This distinction — between *text existing* and *target data being in that text* — is the central problem documented in [[summaries/FIX_EXPLANATION]].

A further upstream challenge: many clinical PDFs are **scanned documents** with no usable text layer at all. Before any score extraction can occur, these PDFs must be processed through an [[concepts/ocr-pipeline]] to generate a clean text layer. This makes OCR pre-processing a prerequisite step in many real-world workflows.

The key insight: **a PDF assessment function must answer the right question.** Generic text-layer checks (e.g., character count thresholds) are insufficient for clinical score extraction.

---

## Common Use Cases

- Bulk document processing: extracting numbers, dates, and key information from a folder of PDFs into a spreadsheet
- Converting unstructured clinical reports into structured tables for database ingestion
- Automating data entry from scanned or digital neuropsychological assessment documents
- Feeding extracted scores into downstream narrative generation or reporting pipelines

---

## PDF Score Formats

Clinical assessment PDFs typically encode scores in one of three ways:

### 1. Graphical / Bar Chart Format
- Scores are rendered as visual bar charts or profile graphs
- The text layer contains no score values
- **Not programmatically extractable** without image-based methods (e.g., OCR on rendered images, computer vision)
- Example: `AS_PAI_Report.pdf` — 18,242 characters of text (headers, prose, footnotes), but T-scores embedded in bar charts on pages 3–4

### 2. Table / Structured Text Format
- Scores appear in tabular form within the text layer
- Parseable with regex or structured text extraction
- **Programmatically extractable**
- Example: PAI Summary Table PDFs

### 3. Inline Prose Format
- Scores mentioned within narrative text (e.g., "a T-score of 72")
- Extractable with pattern matching or NLP techniques, but fragile

---

## OCR Pre-Processing

Before score extraction is possible, scanned PDFs must pass through an OCR stage. The [[concepts/ocr-pipeline]] concept covers this in detail. In brief:

### Assessing PDF Text Quality

A three-tier classification helps route PDFs appropriately:

| Status | Character Count | Action |
|---|---|---|
| `NO_TEXT` | < 100 chars | Requires OCR |
| `POOR_TEXT` | < 1,000 chars | May need OCR |
| `GOOD_TEXT` | ≥ 1,000 chars | Ready for extraction |

```r
check_pdf_text <- function(pdf_path) {
  text <- pdftools::pdf_text(pdf_path)
  total_chars <- sum(nchar(text))
  if (total_chars < 100) return("NO_TEXT - needs OCR")
  else if (total_chars < 1000) return("POOR_TEXT - may need OCR")
  else return("GOOD_TEXT - ready for extraction")
}
```

### OCR Tool Selection

For clinical PDF workflows, **OCRmyPDF** is the recommended tool for batch processing:
- Adds a clean OCR text layer while preserving PDF format
- Handles existing text layers gracefully (`--skip-text`)
- Supports deskewing, cleaning, and page rotation
- Free and open source; installable via `brew install ocrmypdf`

For documents with complex layouts (tables, multi-column), Google Cloud Vision API offers superior accuracy but at per-page cost.

For selective R-native processing, the `tesseract` + `pdftools` combination converts pages to images at 300 DPI, OCRs each page, and saves output as markdown.

### Pre-Processing Parameters
- **Minimum DPI:** 300 for acceptable quality
- **Recommended DPI:** 400 for small clinical text
- **Pre-processing steps:** deskew, artifact cleaning, page rotation, margin cropping

### Integration Pattern

```r
# Step 0: OCR if needed
if (check_pdf_text(pdf_path) != "GOOD_TEXT - ready for extraction") {
  pdf_path <- ocr_single_pdf(pdf_path)  # returns path to OCR'd PDF
}

# Step 1: Assess extractability
res <- assess_pai_pdf(pdf_path)

# Step 2: Extract or route to manual entry
if (res$scores_extractable) {
  extraction <- extract_pai_scores(pdf_path)
} else {
  # Prompt user with page-specific guidance
}
```

---

## The Two-Question Framework

A robust extraction pipeline must distinguish between two separate questions:

| Question | Tool | Answers |
|---|---|---|
| Does this PDF have a text layer? | `assess_pdf_text()` | YES / NO |
| Are the target scores in that text? | `assess_pai_pdf()` | YES / NO |

Confusing these questions causes false confidence — reporting a file as "extractable" when its scores are actually graphical. This is exactly the bug described in [[summaries/FIX_EXPLANATION]]. The OCR quality check is a necessary precondition to both questions.

---

## Assessment Function Design

A well-designed PDF score assessment function (e.g., `assess_pai_pdf()`) should return:

- **Has text layer**: boolean
- **Is recognized report type**: boolean (e.g., is this a PAI report?)
- **Are target scores in text**: boolean
- **Extraction method**: one of `TABLE_EXTRACTION`, `GRAPHICAL_ONLY`, `NOT_A_REPORT`, etc.
- **Needs manual entry**: boolean

This replaces a simple `GOOD_TEXT / POOR_TEXT / NO_TEXT` status with actionable, domain-specific guidance.

---

## Extraction Methods

### Automatic (Table Extraction)
- Applicable when scores appear in structured text
- Use PDF text parsing libraries (e.g., `pdftools` in R)
- Match score patterns with regex
- Return structured score objects
- Works with: PAI Summary Table PDFs, some other report formats

### Spreadsheet Output
- Extracted numbers, dates, and key information are organized into rows and columns
- Suitable for CSV or Excel export
- Enables bulk data entry automation across a folder of PDFs
- Column schema should be defined in advance for consistency across documents

### Manual Entry Required
- Applicable when scores are graphical only
- System should guide the user: which pages to open, what to look for
- Example guidance: "Open pages 3–4 and enter T-scores from bar graphs"
- No false promise of automation

### Hybrid / OCR-Based
- Render PDF pages as images at 300–400 DPI, then apply OCR or computer vision to detect bar heights or labeled values
- High complexity; not always reliable for clinical precision
- Falls under the broader [[concepts/ocr-pipeline]] and [[concepts/clinical-pdf-assessment]] strategies
- Tools: Tesseract 5.0+ (via `tesseract` R package), OCRmyPDF, or Google Cloud Vision for complex layouts

---

## Relationship to PAI Assessment

The PAI (Personality Assessment Inventory) is a specific case study for this problem. See [[concepts/pai-assessment]] for the broader clinical context. PAI reports from commercial scoring software commonly render T-score profiles as graphical bar charts, making automated extraction impossible without specialized handling.

The `extract_pai_scores()` function described in [[summaries/FIX_EXPLANATION]] auto-detects PAI PDF type and routes to the appropriate extraction method.

---

## Implementation Pattern (R)

```r
# Step 0: Ensure clean text layer
if (check_pdf_text(pdf_path) != "GOOD_TEXT - ready for extraction") {
  pdf_path <- ocr_single_pdf(pdf_path)
}

# Step 1: Assess
res <- assess_pai_pdf(pdf_path)

# Step 2: Branch on extractability
if (res$scores_extractable) {
  extraction <- extract_pai_scores(pdf_path)
  if (extraction$success) {
    # Auto-populate score template or export to spreadsheet
  }
} else {
  # Prompt user for manual entry
  # Provide page-specific guidance
}
```

This pattern cleanly separates OCR pre-processing, assessment, and extraction from UI feedback — a key principle for maintainable [[concepts/neuropsychological-assessment-pipeline]] code.

---

## Failure Modes to Avoid

| Failure | Cause | Fix |
|---|---|---|
| False "extractable" status | Generic text check, not score-specific check | Use domain-specific assessment function |
| Silent extraction failure | No success/failure reporting | Always return structured result with `success` flag |
| Wrong extraction method | Not detecting PDF subtype | Auto-detect report type before extracting |
| User confusion | No guidance when manual entry needed | Return actionable message with page references |
| OCR skipped on poor scans | Assuming any text = good text | Always run `check_pdf_text()` first; OCR if below threshold |
| Poor OCR quality | Low DPI or unskewed scans | Use ≥300 DPI, `--deskew`, `--clean` flags |
| Unstructured folder input | No schema defined before bulk extraction | Define column schema and target data types upfront |

---

## Related Concepts

- [[concepts/ocr-pipeline]] — Pre-processing step to generate text layers from scanned PDFs
- [[concepts/clinical-pdf-assessment]] — Broader framework for evaluating clinical PDFs
- [[concepts/neuropsychological-test-scores]] — The score types being extracted
- [[concepts/neuropsychological-assessment-pipeline]] — End-to-end workflow context
- [[concepts/pai-assessment]] — PAI-specific clinical context
- [[concepts/fallback-strategy]] — Handling cases where automatic extraction fails
- [[concepts/phi-data-handling]] — Privacy considerations when processing score PDFs
- [[concepts/r-python-integration]] — Tooling choices for PDF parsing implementations

See also: [[summaries/extract_pdf]], [[summaries/OCR_PDF_GUIDE]], [[summaries/FIX_EXPLANATION]], [[summaries/KNOWLEDGE_BASE_EXPLAINED]], [[summaries/QUICK_REFERENCE]], [[summaries/README_PIPELINE]], [[summaries/README_WORKFLOW]], [[summaries/WORKFLOW_INSTRUCTIONS]], [[summaries/neuropsych-data-extractor]], [[summaries/neuropsych-pdf-parser]], [[summaries/text-extraction]], [[summaries/full-pipeline]], [[summaries/SESSION_SUMMARY_2025-04-28]]

See also: [[summaries/2026-02-11-this-session-is-being-continued-from-a-previous-co]]

See also: [[summaries/AGE_OVERRIDE_GUIDE]]