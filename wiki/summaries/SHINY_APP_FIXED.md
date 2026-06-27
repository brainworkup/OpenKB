---
doc_type: short
full_text: sources/SHINY_APP_FIXED.md
---

# Shiny App Fix: PAI PDF Assessment Logic

**Date:** January 29, 2026
**Status:** Fixed and tested

## Overview

This document records a bug fix to a Shiny app used for assessing and extracting T-scores from [[concepts/pai-assessment]] PDFs (Personality Assessment Inventory). The core issue was that the app incorrectly classified graphical PAI Score Reports as extractable.

## Problem & Root Cause

The `assess_pai_pdf()` function used overly strict criteria — requiring ≥15 scale names to be found in the first 3 pages of text. However, PAI Score Reports place scale names only in profile pages (not in early text pages), causing misclassification.

**Result before fix:** `AS_PAI_Report.pdf` was flagged as "extractable" when T-scores were actually embedded in bar charts (graphical format).

## Solution

Updated detection logic in `assess_pai_pdf.R`:
1. Detect keywords "Score Report" or "Full Scale Profile" in text.
2. Check whether T-score numbers are actually present as text.
3. If no T-score numbers found → classify as `GRAPHICAL_ONLY`.

## PAI PDF Classification Types

| Type | Extraction Method | Action Required |
|---|---|---|
| Score Reports (graphical) | `GRAPHICAL_ONLY` | Manual T-score entry |
| Summary Tables | `TABLE_EXTRACTION` | Automatic extraction |
| Scanned/No Text | OCR required | Run OCR pipeline |

## Files Modified

- **`assess_pai_pdf.R`** — Fixed GRAPHICAL_ONLY detection logic
- **`app.R`** — Replaced with fixed version
- **`app_BACKUP_*.R`** — Original app backed up

## Next Steps for AS Patient

Three options for manual T-score entry when `GRAPHICAL_ONLY` is detected:
1. **Interactive Entry** — `source("process_AS_complete.R")` prompts for each T-score, saves to JSON, generates interpretation via [[concepts/retrieval-augmented-generation]].
2. **Template Editing** — Edit `input/AS_scores_template.json` then run the processor.
3. **Demo Mode** — `source("demo_AS_with_example_scores.R")` uses example scores.

## Verification

```r
source("assess_pai_pdf.R")
test <- assess_pai_pdf("input/AS_PAI_Report.pdf")
test$extraction_method   # "GRAPHICAL_ONLY"
test$needs_manual_entry  # TRUE
```

## Key Concepts

- [[concepts/pai-assessment]] — PAI PDF formats and T-score extraction challenges
- [[concepts/pdf-score-extraction]] — Methods for extracting T-scores from PDFs
- [[concepts/ocr-pipeline]] — OCR processing used as fallback for scanned PDFs
- [[concepts/clinical-pdf-assessment]] — Broader framework for clinical PDF classification
- [[concepts/neuropsychological-test-scores]] — T-scores and their role in neuropsychological reporting
- [[concepts/retrieval-augmented-generation]] — RAG-based interpretation pipeline used downstream

## Related Concepts
- [[concepts/neuropsychological-assessment-pipeline]]
