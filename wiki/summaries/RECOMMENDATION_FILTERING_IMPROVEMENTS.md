---
doc_type: short
full_text: sources/RECOMMENDATION_FILTERING_IMPROVEMENTS.md
---

# Summary: Recommendation Content Filtering Improvements

## Overview

This document describes enhancements made to `src/report_parser.py` to improve the quality of extracted recommendations from clinical/psychological reports. The core problem was that non-actionable content — clinical assessments, administrative notes, and signature blocks — was being incorrectly captured alongside genuine, actionable recommendations.

## Problem

Three categories of unwanted content were being extracted as recommendations:

1. **Clinical Assessment/Impressions** — Interpretive paragraphs describing patient characteristics, motivation, and prognosis (e.g., "Ms. [PATIENT] appears to have substantial interest in making changes...").
2. **Administrative Notes** — Billing hours, contact information (e.g., "A total of 8 hours was spent by me personally...").
3. **Signature Blocks** — Phone numbers and contact details appearing after the recommendations section ends.

## Changes Made

### 1. Enhanced Section End Detection
Added regex patterns to terminate extraction when administrative or signature content is encountered:
- Phone number patterns: `Please call (###) ###-####`
- Billing/hour statements: `A total of N hours was spent`
- Personal attestation language: `I personally performed`

### 2. Content Filtering Function — `_filter_non_recommendation_content()`
A new function filters out paragraphs identified as assessment or administrative content based on language markers:

**Filtered out (assessment language):**
- "appears to have", "is probably pessimistic", "with respect to past", "may have initial difficulty", "furthermore, concerns about"
- Clinical impressions, prognostic statements, treatment readiness assessments

**Filtered out (administrative):**
- Hour/billing disclosures
- Contact/phone information

**Kept (actionable recommendations):**
- Accommodations, referrals, therapeutic strategies
- Educational recommendations and specific interventions

### 3. Integration into Subsection Splitting
Filtering is automatically applied at the start of `split_recommendations_by_subsection()`, making it transparent to downstream consumers.

## Impact

| Aspect | Before | After |
|---|---|---|
| Content quality | Mixed with assessment paragraphs | Only actionable recommendations |
| Administrative noise | Included | Removed |
| [[concepts/retrieval-augmented-generation]] suitability | Lower | Higher |
| Actionability | Hard to identify | Clear and focused |

## Testing Approach

Post-ingestion validation by inspecting `data/recommendations_kb.json` for the previously problematic file (`amoozegar_report_081018.pdf`) to confirm absence of assessment and administrative language.

## Extensibility

- Additional `assessment_markers` can be appended to the filter list as new patterns are discovered.
- Future enhancements proposed: machine learning classification for recommendation vs. assessment, configurable filters, confidence scoring, and manual review flagging.

## Related Concepts
- [[concepts/neuropsych-report-parsing]]
- [[concepts/clinical-nlp-pipelines]]
- [[concepts/phi-deidentification-pipeline]]

This work directly improves the [[concepts/report-parser-quality]] of the [[concepts/neuropsychological-assessment-pipeline]] and enhances the usefulness of the [[concepts/recommendation-rag-pipeline]]. It is part of broader [[concepts/report-ingestion-pipeline]] quality control, and the filtered output feeds into [[concepts/rag-chunking]] and [[concepts/text-chunking]] workflows for embedding and retrieval. The [[concepts/clinical-text-classification]] challenge of distinguishing assessment language from actionable recommendations is central to this improvement.

## Compatibility

All changes are backward compatible — filtering is purely additive (removes unwanted content, preserves wanted content) with no structural changes to output data.

## Files Modified

- `src/report_parser.py` (lines ~1145–1216, ~1230)
  - Added phone/admin stop patterns
  - Added `_filter_non_recommendation_content()` function
  - Integrated filtering into `split_recommendations_by_subsection()`