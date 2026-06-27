---
doc_type: short
full_text: sources/FIX_EXPLANATION.md
---

# FIX_EXPLANATION — PAI Score Extraction: Problem & Solution

**Date:** January 29, 2026

## Overview

This document diagnoses a critical disconnect between what the [[concepts/clinical-pdf-assessment]] function (`assess_pdf_text()`) reports and what is actually extractable from a PAI report PDF. It introduces two new R functions to correctly identify and handle PAI T-score extraction scenarios.

---

## The Core Problem

Two systems gave contradictory answers about the same file (`AS_PAI_Report.pdf`):

- **Shiny App** (using `assess_pdf_text()`): Reported the PDF as `GOOD_TEXT` — implying T-scores are extractable.
- **Databot**: Could not find T-scores — correctly determined they are embedded in bar chart graphics.

The root cause is that `assess_pdf_text()` answers the wrong question. It checks *whether a text layer exists*, not *whether PAI T-scores are present in that text layer*.

### What `assess_pdf_text()` Checks vs. Misses

| Checks | Does NOT Check |
|---|---|
| Total character count | Whether PAI T-scores are in the text |
| Whether a text layer exists | Whether scores are in tables vs. graphics |
| | Whether the PDF is a PAI report at all |

The `AS_PAI_Report.pdf` has 18,242 characters of genuine text (headers, demographics, interpretive prose, footnotes), but the **T-scores themselves are embedded in graphical bar charts on pages 3–4** — not in the text layer.

---

## The Solution

### New Functions

**`assess_pai_pdf()` (`assess_pai_pdf.R`)**
A [[concepts/pai-assessment]]-specific function that answers PAI-specific questions:
- Does the PDF have a text layer?
- Is it a PAI report?
- Are PAI T-scores present in the text?
- What extraction method is appropriate?
- Is manual entry required?

**`extract_pai_scores()` (`extract_pai_scores.R`)**
An intelligent [[concepts/pdf-score-extraction]] function that:
- Auto-detects PDF type (Score Report vs. Summary Table)
- Uses the appropriate extraction method
- Returns structured T-scores
- Reports success/failure clearly

---

## Test Results Comparison

### Old Assessment (`assess_pdf_text`)
```
Status: GOOD_TEXT | Characters: 18,242 | Needs OCR: FALSE
```
→ Misleadingly suggests scores are extractable.

### New Assessment (`assess_pai_pdf`)
```
Has text: TRUE (GOOD_TEXT)
Is PAI report: TRUE
T-scores extractable: FALSE
Extraction method: GRAPHICAL_ONLY
Needs manual entry: TRUE
```
→ Correctly identifies the situation.

---

## Shiny App Fix

### Option 1: Quick Fix
Replace line 136 in `app.R`:
```r
# OLD:
res <- assess_pdf_text(files$datapath[i])
# NEW:
res <- assess_pai_pdf(files$datapath[i])
```
Also requires sourcing `assess_pai_pdf.R` alongside `ocr_utils.R`.

### Option 2: Enhanced Version
After assessment, conditionally call `extract_pai_scores()` when `res$scores_extractable` is `TRUE`, auto-populating the score template with user review before finalizing.

---

## Expected Behavior After Fix

| PDF Type | Assessment Result | User Guidance |
|---|---|---|
| Graphical scores (e.g., AS_PAI_Report.pdf) | ⚠️ GRAPHICAL_ONLY | Open pages 3–4, enter T-scores manually |
| Summary Table PDFs | ✅ TABLE_EXTRACTION | Click to extract automatically |

---

## Key Concepts

- [[concepts/clinical-pdf-assessment]] — The general problem of assessing PDF content quality
- [[concepts/pai-assessment]] — PAI-specific logic for determining score extractability
- [[concepts/pdf-score-extraction]] — Methods for extracting PAI T-scores from different PDF formats
- [[concepts/neuropsychological-test-scores]] — Distinction between T-scores embedded in graphics vs. text layers
- [[concepts/fallback-strategy]] — Determining when automatic extraction must fall back to manual entry

---

## Files Created

| File | Purpose |
|---|---|
| `assess_pai_pdf.R` | PAI-specific PDF assessment |
| `extract_pai_scores.R` | Intelligent T-score extraction |
| `FIX_EXPLANATION.md` | This explanation document |

---

## Bottom Line

The Shiny app was "too optimistic" about extractable text. The fix is a single-line change in `app.R` (line 136) to swap `assess_pdf_text()` for `assess_pai_pdf()`, giving users accurate guidance about whether automatic extraction or manual entry is required.

## Related Concepts
- [[concepts/neuropsychological-assessment-pipeline]]
- [[concepts/pai-knowledge-base]]
- [[concepts/neuropsychological-reporting]]
- [[concepts/r-python-integration]]
