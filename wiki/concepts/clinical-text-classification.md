---
sources: [summaries/bwu.neuro.reports.recs.adhd.merge.md, summaries/RECOMMENDATION_FILTERING_IMPROVEMENTS.md]
brief: Distinguishing actionable clinical recommendations from assessment narratives and administrative text in reports.
---

# Clinical Text Classification: Assessment vs. Recommendation

A core challenge in [[concepts/neuropsych-report-parsing]] and [[concepts/report-ingestion-pipeline]] is that clinical documents interleave several distinct types of text — actionable recommendations, interpretive assessments, and administrative boilerplate — within the same section. Correctly separating these categories is essential for downstream tasks such as [[concepts/recommendation-rag-pipeline]], embedding quality, and [[concepts/retrieval-augmented-generation]].

## The Core Distinction

### Actionable Recommendations
Text that tells a clinician, educator, or patient what to *do*. Characteristics:
- Imperative or directive phrasing: "Consider referral to...", "Extended time accommodations are apt to..."
- Specifies interventions, accommodations, strategies, or referrals
- Addresses future actions
- Oriented toward treatment, education, or support planning

### Clinical Assessment / Impressions
Text that describes the *patient* — their characteristics, motivation, prognosis, or test-derived impressions. Characteristics:
- Descriptive, third-person language about the patient: "She appears to have...", "Her responses indicate..."
- Prognostic statements: "may have initial difficulty..."
- Interpretive commentary: "With respect to past reported suicidal ideation..."
- Not directly actionable

### Administrative / Signature Content
Boilerplate text appended after clinical content ends:
- Billing hour disclosures: "A total of 8 hours was spent by me personally..."
- Contact information: "Please call (323) 442-4058 with any questions..."
- Attestation language: "I personally performed all of the above..."

## Why This Matters

When clinical reports are parsed for knowledge bases or RAG retrieval, unfiltered extraction captures all three categories. This degrades:
- **Embedding quality**: Assessment and administrative text dilutes the semantic signal of actionable recommendations
- **Retrieval precision**: Queries for "what accommodations are recommended" may surface patient impressions instead
- **Downstream generation**: LLM prompts populated with mixed content produce less focused outputs

See [[summaries/RECOMMENDATION_FILTERING_IMPROVEMENTS]] for a concrete case from `amoozegar_report_081018.pdf` where all three unwanted categories appeared inside a recommendations section.

## Rule-Based Classification Approaches

The approach documented in [[summaries/RECOMMENDATION_FILTERING_IMPROVEMENTS]] uses pattern matching at two levels:

### 1. Section Boundary Detection (Stop Patterns)
Regex patterns that signal the *end* of recommendation content and the beginning of administrative content:
```
Please call (###) ###-####
A total of N hours was spent
I personally performed
Please do not hesitate to contact
```
These act as hard stops during text extraction in [[concepts/clinical-report-structure]] parsing.

### 2. Paragraph-Level Filtering (Assessment Markers)
A list of phrasal markers that identify assessment/interpretive paragraphs:
- `"appears to have"` — motivation/characteristic impressions
- `"is probably pessimistic"` — prognostic characterization
- `"with respect to past"` — history framing
- `"furthermore, concerns about"` — clinical concern enumeration
- `"may have initial difficulty"` — treatment readiness caveats
- `"her responses to questions indicate"` — test-derived impressions

Paragraphs beginning with these markers are filtered before subsection splitting.

## Integration into the Pipeline

In `src/report_parser.py`, filtering is applied *before* subsection splitting in `split_recommendations_by_subsection()`, making it transparent to all downstream consumers. This follows a clean-input-first principle: garbage is removed at ingestion, not at query time.

This relates to broader concerns in [[concepts/clinical-nlp-pipelines]] and [[concepts/report-parser-quality]] about ensuring that only semantically appropriate content reaches the vector store.

## Extensibility and Limitations

### Extending the Rule Set
New assessment patterns can be appended to `assessment_markers` as new report authors or formats are encountered. This is an open-ended, corpus-driven process.

### Limitations of Rule-Based Classification
- **False negatives**: Novel phrasing not in the marker list passes through unfiltered
- **False positives**: Rare cases where assessment-flavored language appears inside a genuine recommendation
- **Author variability**: Different clinicians write recommendations differently; no single marker set covers all styles

### Future Directions
The [[summaries/RECOMMENDATION_FILTERING_IMPROVEMENTS]] document anticipates moving toward:
- **Machine learning classifiers** trained on labeled recommendation vs. assessment paragraphs (see [[concepts/clinical-text-classification]])
- **Confidence scoring** to tag borderline paragraphs
- **Configurable filter profiles** per report author or institution
- **Manual review flagging** for human adjudication of uncertain cases

## Related Concepts

- [[concepts/neuropsych-report-parsing]] — Broader parsing pipeline this classification fits within
- [[concepts/report-parser-quality]] — Quality assurance for parsed report content
- [[concepts/recommendation-rag-pipeline]] — Downstream consumer of filtered recommendations
- [[concepts/retrieval-augmented-generation]] — End use case that benefits from clean classification
- [[concepts/clinical-report-structure]] — Document structure that governs section boundaries
- [[concepts/rag-chunking]] — How filtered text is subsequently chunked for vector storage
- [[concepts/text-chunking]] — General chunking strategies applied after filtering
- [[concepts/report-ingestion-pipeline]] — Full pipeline in which this classification step participates


See also: [[summaries/bwu.neuro.reports.recs.adhd.merge]]