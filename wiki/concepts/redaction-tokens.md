---
sources: [summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_PIPELINE.md, summaries/clinical-validity-reviewer.md, summaries/SKILL.md]
brief: Redaction tokens are placeholder markers that protect sensitive information in clinical reports during automated processing.
---

# Redaction Tokens

Redaction tokens are structured placeholder markers inserted into clinical text to replace sensitive, private, or identifying information before that text is processed, stored, or rendered. They allow downstream systems — including AI-assisted report writers and templating engines — to handle documents without ever exposing the underlying protected data.

## Role in Neuropsychological Reporting

Within the Luria application, redaction tokens appear in structured findings that feed into the [[concepts/neuropsychological-reporting]] pipeline. The [[summaries/SKILL]] document (`luria-report-writing`) explicitly mandates that these tokens **must be preserved** during report generation. The report-writing skill must not expand, resolve, or strip them — they pass through unchanged into the final rendered output.

This design ensures that:
- Protected health information (PHI) never appears in generated prose
- Reports can be reviewed, edited, or shared at intermediate stages without privacy risk
- Token replacement (i.e., filling in real values) happens only at a controlled, final output stage — if at all

## Relationship to PII and PHI Handling

Redaction tokens are a core mechanism in [[concepts/pii-redaction-pipelines]] and support [[concepts/phi-data-handling]] and [[concepts/privacy-first-software]] design. Rather than scrubbing data after the fact, the token approach prevents sensitive data from entering the processing pipeline in the first place.

## Format Considerations

Because Luria reports target markdown, Typst, and Quarto surfaces (see [[summaries/SKILL]]), redaction tokens must be format-neutral — they should not break rendering in any of these systems. Typical implementations use delimiters such as `{{PATIENT_NAME}}`, `[REDACTED:DOB]`, or similar patterns that are visually distinct and machine-parseable.

## Connections
- [[concepts/phi-data-handling]] — broader framework for managing protected health information
- [[concepts/pii-redaction-pipelines]] — automated pipelines that generate and consume redaction tokens
- [[concepts/privacy-first-software]] — design philosophy under which token-based redaction operates
- [[concepts/neuropsychological-reporting]] — the primary output context where token preservation is enforced
- [[summaries/SKILL]] — source skill definition requiring token preservation in report prose


See also: [[summaries/clinical-validity-reviewer]]

See also: [[summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_PIPELINE]]