---
sources: [summaries/extract_pdf.md, summaries/neuropsych-pdf-parser.md, summaries/SHINY_APP_FIXED.md, summaries/OCR_PDF_GUIDE.md, summaries/FIX_EXPLANATION.md]
brief: Analyzing clinical PDFs to determine content type, machine-readability, and appropriate extraction method for clinical workflows.
---

# Clinical PDF Assessment

Clinical PDF assessment refers to the process of analyzing a PDF document to determine what kind of content it contains, whether that content is machine-readable, and what extraction methods are appropriate for downstream clinical use. It is distinct from generic PDF text detection because it must answer **clinical workflow questions** — not just "does this file have text?" but "does it contain the specific clinical data we need?"

---

## The Generic vs. Clinical Assessment Problem

Standard PDF text detection functions (e.g., `assess_pdf_text()` in R's `pdftools` ecosystem) answer a single question:

> *Does this PDF have a text layer with sufficient character count?*

This is useful for deciding whether OCR is needed, but it is **insufficient for clinical data extraction**. A PAI report PDF may have 18,000+ characters of legitimate text (headers, demographics, interpretive prose) while its actual clinical scores — the T-scores plotted on bar charts — exist only as graphical elements with no text representation.

This gap causes a critical failure mode: the system reports the file as "extractable" when the clinically relevant data cannot be programmatically retrieved. See [[summaries/FIX_EXPLANATION]] for a detailed case study of this exact disconnect, and [[summaries/SHINY_APP_FIXED]] for the corresponding Shiny app bug fix that resolved this misclassification in a real workflow.

---

## A Real-World Example: The PAI Score Report Fix

The `AS_PAI_Report.pdf` case illustrates the clinical assessment problem precisely. The `assess_pai_pdf()` function originally required ≥15 scale names to be found in the first 3 pages of text. Because PAI Score Reports place scale names only in profile pages (not early text pages), the function misclassified the file as unclear/extractable.

**The fixed logic:**
1. Detect "Score Report" or "Full Scale Profile" keywords in text
2. Check whether T-score numbers are actually present as text characters
3. If no T-score numbers found → classify as `GRAPHICAL_ONLY`

**Before fix:**
```
❓ AS_PAI_Report.pdf - PAI report but extraction method unclear
Assessment: 0 need manual entry, 0 need OCR
```

**After fix:**
```
⚠️  AS_PAI_Report.pdf - GRAPHICAL scores, manual entry required
Assessment: 1 need manual entry
Extraction method: GRAPHICAL_ONLY
Needs manual entry: TRUE
Confidence: high
```

This demonstrates that instrument-specific keyword detection and score-presence verification are essential components of a reliable clinical assessment function.

---

## Step 0: OCR Pre-Assessment

Before clinical assessment can proceed, a PDF must have a usable text layer. A practical threshold approach (as codified in the OCR PDF Processing Guide) uses character counts from `pdftools::pdf_text()`:

| Character Count | Status | Action |
|---|---|---|
| < 100 | `NO_TEXT` | OCR required before any clinical assessment |
| 100–1,000 | `POOR_TEXT` | OCR recommended; clinical assessment unreliable |
| > 1,000 | `GOOD_TEXT` | Proceed to clinical assessment |

This pre-assessment step feeds directly into the [[concepts/ocr-pipeline]]: documents classified as `NO_TEXT` or `POOR_TEXT` must be routed through an OCR tool (OCRmyPDF, Tesseract, Adobe Acrobat Pro, or Google Cloud Vision) before the clinical dimensions below can be evaluated.

Key insight: a `GOOD_TEXT` classification is **necessary but not sufficient** for clinical score extraction. A document can be text-rich but still store its scores as graphical bar charts — exactly the PAI Score Report scenario described above.

---

## Dimensions of Clinical PDF Assessment

A robust clinical PDF assessment function must evaluate multiple independent dimensions:

### 1. Text Layer Presence
- Does the PDF have a text layer at all?
- How many characters are present?
- Is character count above a meaningful threshold (e.g., >1,000 for `GOOD_TEXT`)?

### 2. Document Identity
- Is this the expected document type (e.g., a PAI report, a neuropsychological battery summary)?
- Can header/footer patterns confirm the instrument or publisher?
- Are instrument-specific keywords present (e.g., "Score Report", "Full Scale Profile")?

### 3. Score Extractability
- Are the target clinical scores (e.g., T-scores, scaled scores, percentiles) present in the text layer?
- Are scores in tables, prose, or graphical elements?
- What extraction method is appropriate?

### 4. Extraction Method Classification

| Classification | Meaning | Action |
|---|---|---|
| `TABLE_EXTRACTION` | Scores in structured text/table | Automatic extraction possible |
| `PROSE_EXTRACTION` | Scores embedded in sentences | NLP extraction may work |
| `GRAPHICAL_ONLY` | Scores in bar charts/images only | Manual entry required |
| `NO_TEXT` | Scanned image, no text layer | OCR required first |

### 5. Manual Entry Flag
- Should the user be prompted to enter scores by hand?
- Can the system provide page-level guidance (e.g., "See pages 3–4")?
- Does the downstream workflow support interactive or template-based entry?

---

## Assessment Output Schema

A well-designed clinical PDF assessment function returns structured metadata, not just a status string:

```r
list(
  has_text = TRUE,
  text_status = "GOOD_TEXT",
  is_target_report = TRUE,
  scores_extractable = FALSE,
  extraction_method = "GRAPHICAL_ONLY",
  needs_manual_entry = TRUE,
  confidence = "high",
  guidance = "Open pages 3-4 and enter T-scores from bar graphs"
)
```

Contrast this with the minimal output of a generic text assessor:

```r
list(
  status = "GOOD_TEXT",
  needs_ocr = FALSE
)
```

The generic output is accurate but clinically misleading. The `confidence` field is particularly important when detection relies on heuristics: a high-confidence `GRAPHICAL_ONLY` classification should trigger immediate manual entry routing, while a lower-confidence classification may warrant a human review step.

---

## Integration with Clinical UI (Shiny)

Clinical PDF assessment logic is most useful when surfaced in a clinical workflow UI. The [[summaries/SHINY_APP_FIXED]] document describes an R Shiny application that wraps `assess_pai_pdf()` and presents assessment results as actionable guidance:

- **⚠️ GRAPHICAL_ONLY** → "Manual entry required from profile pages"
- **✅ TABLE_EXTRACTION** → Automatic T-score extraction runs
- **❌ OCR required** → "Run OCR & Import" button becomes active

This pattern — assessment driving UI state — is a clean separation of concerns: the assessment function determines *what is true about the document*, and the UI determines *what action to surface to the user*.

Verification after a fix can be done programmatically:

```r
source("assess_pai_pdf.R")
test <- assess_pai_pdf("input/AS_PAI_Report.pdf")
test$extraction_method   # Should be "GRAPHICAL_ONLY"
test$needs_manual_entry  # Should be TRUE
```

---

## OCR Method Selection for Clinical Documents

When a clinical PDF fails the text pre-assessment, the choice of OCR method matters for downstream clinical reliability. For neuropsychological assessment documents:

- **OCRmyPDF** (using Tesseract 5.0+) is recommended for batch processing of assessment folders. It preserves the PDF format so that subsequent clinical extraction tools (`pdftools`, `ragnar`) can read the result directly. Key flags: `--deskew`, `--clean`, `--rotate-pages`, `--skip-text`.
- **Adobe Acrobat Pro** offers the highest quality for individual high-value documents.
- **Tesseract via R** (`tesseract` + `pdftools` packages) is useful in custom R-native pipelines that output Markdown text files rather than PDFs.
- **Google Cloud Vision API** handles complex layouts (multi-column reports, embedded tables) but raises [[concepts/phi-data-handling]] concerns: clinical PDFs often contain Protected Health Information, and assessment functions must not transmit document content to external services without appropriate data agreements.

A minimum of 300 DPI is required for reliable OCR; 400 DPI is recommended when reports contain small-font score tables.

After OCR, a validation step should confirm the character count has increased meaningfully before proceeding to clinical assessment proper.

---

## Full Assessment Pipeline

```
[PDF arrives]
     ↓
[1. Text pre-assessment: count chars]
     ↓
 [NO_TEXT / POOR_TEXT] → [OCR pipeline] → [re-assess]
 [GOOD_TEXT] ↓
[2. Document identity check]
     ↓
[3. Score extractability check: keyword + numeric scan]
     ↓
[4. Route: TABLE | PROSE | GRAPHICAL | MANUAL]
     ↓
[5. Surface in UI with actionable guidance]
```

This pipeline structure ensures that OCR and clinical extraction are **decoupled phases**, each with their own success criteria and logging. Step 3 is the critical addition over naive text detection: it verifies not just that text exists, but that the *clinically relevant* text (score numbers) exists.

---

## Relationship to Fallback Strategy

Clinical PDF assessment is the entry point of a [[concepts/fallback-strategy]]: depending on what the assessment finds, the pipeline routes to automatic extraction, semi-automatic extraction with user review, or a fully manual entry workflow. Getting the assessment right is prerequisite to every downstream step.

For the `GRAPHICAL_ONLY` case, downstream options typically include:
1. **Interactive entry** — prompt the user for each T-score in sequence
2. **Template-based entry** — provide a JSON template for the user to populate offline
3. **Demo/example mode** — run the pipeline with example scores for testing

---

## Relationship to Score Extraction

Assessment and extraction are separate concerns. Assessment answers *whether* and *how* extraction is possible; [[concepts/pdf-score-extraction]] answers *how to do it* for a given document type. A well-architected pipeline keeps these phases decoupled so that assessment results can be logged, audited, and used to route work — independently of whether extraction is actually attempted.

---

## Clinical Context

In [[concepts/neuropsychological-assessment-pipeline]] workflows, PDFs arrive from many sources: test publishers (e.g., PAI, WAIS, MMPI), hospital systems, and clinician-generated summaries. Each source has different formatting conventions, and scores may appear in radically different positions and formats within the text layer — or not at all. Clinical PDF assessment must be instrument-aware, not just text-aware.

This connects to broader concerns in [[concepts/phi-data-handling]]: clinical PDFs often contain Protected Health Information, and assessment functions must not transmit document content to external services during the classification step. This constraint rules out cloud OCR services like Google Cloud Vision for most clinical workflows unless explicit data agreements are in place.

---

## Related Pages

- [[summaries/FIX_EXPLANATION]] — Case study of `assess_pdf_text()` vs. `assess_pai_pdf()` for a PAI report
- [[summaries/SHINY_APP_FIXED]] — Shiny app fix: correct GRAPHICAL_ONLY detection for PAI Score Reports
- [[summaries/OCR_PDF_GUIDE]] — Full reference for OCR tools, R integration code, and batch workflows
- [[concepts/ocr-pipeline]] — OCR tool selection, pre-processing, and quality validation
- [[concepts/pdf-score-extraction]] — Methods for extracting clinical scores once extractability is confirmed
- [[concepts/pai-assessment]] — PAI-specific assessment context
- [[concepts/neuropsychological-assessment-pipeline]] — The broader pipeline this feeds into
- [[concepts/neuropsychological-test-scores]] — The clinical data types being assessed for
- [[concepts/fallback-strategy]] — How assessment results route downstream workflows
- [[concepts/phi-data-handling]] — Privacy constraints on PDF processing

See also: [[summaries/neuropsych-pdf-parser]]

See also: [[summaries/extract_pdf]]