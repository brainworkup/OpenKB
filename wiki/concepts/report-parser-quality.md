---
sources: [summaries/SESSION_SUMMARY.md, summaries/RECOMMENDATION_FILTERING_IMPROVEMENTS.md, summaries/QUICK_REFERENCE.md, summaries/DIAGNOSIS_PARSER_IMPROVEMENTS.md]
brief: Systematic measurement and improvement of clinical report parser accuracy, coverage, and content filtering quality.
---

# Report Parser Quality and Coverage

Report parser quality and coverage refers to the systematic measurement, diagnosis, and improvement of how completely and accurately an automated parser extracts structured data from clinical documents — particularly neuropsychological reports. A high-quality parser minimizes the number of reports that yield empty or incomplete structured output despite containing parseable content, and ensures that extracted content is genuinely actionable rather than contaminated by assessment language or administrative noise.

## Why Coverage and Content Quality Matter

In clinical NLP pipelines, a report with zero extracted diagnoses is often indistinguishable from a report that genuinely lacks a diagnostic section. Without active quality monitoring, silent extraction failures accumulate and silently degrade the downstream knowledge base. In the autism-RAG project, **24 of 104 reports (23%)** initially had zero diagnoses extracted — a coverage gap that was only discovered through explicit auditing.

Beyond coverage, *content quality* matters equally: even when extraction succeeds, the extracted text may include non-actionable clinical assessment paragraphs, administrative billing notes, or signature blocks. These contaminate recommendation knowledge bases and degrade RAG retrieval quality.

See [[summaries/DIAGNOSIS_PARSER_IMPROVEMENTS]] for the full account of how coverage gaps were identified and addressed, and [[summaries/RECOMMENDATION_FILTERING_IMPROVEMENTS]] for how recommendation content filtering was improved.

## Coverage Metrics

Key metrics to track for parser quality:

| Metric | Description |
|---|---|
| **Zero-extraction rate** | % of real patient reports yielding no structured output |
| **False negatives** | Reports with content that the parser missed |
| **False positives** | Spurious extractions (e.g., subsection labels parsed as diagnoses) |
| **Field completeness** | % of reports with each field populated (diagnoses, age, scores, etc.) |
| **Content contamination rate** | % of extracted recommendations containing assessment or administrative text |

A healthy pipeline distinguishes between *expected* zero-extraction (textbook samples, recommendations-only documents) and *unexpected* zero-extraction (real patient reports with parseable sections).

## Root Causes of Low Quality

### 1. Incomplete Section Header Vocabularies
Clinical reports use diverse, non-standardized section names. A parser that only recognizes "Diagnostic Impressions" will miss "DIAGNOSTIC CONSIDERATIONS", 

See also: [[summaries/SESSION_SUMMARY]]