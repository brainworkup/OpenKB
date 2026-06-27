---
sources: [summaries/DIAGNOSIS_FIX_SUMMARY.md, summaries/extract_pdf.md]
brief: Automated extraction of structured data (numbers, dates, key info) from PDF documents into spreadsheets.
---

# PDF Data Extraction

PDF Data Extraction refers to the automated process of parsing PDF files to retrieve structured information — such as numbers, dates, names, and key data points — and organizing that output into a usable format like a spreadsheet or database table.

## Core Concept

PDFs are inherently unstructured: their content is encoded for visual presentation rather than data access. Extracting meaningful information requires dedicated parsing strategies that can handle both native digital PDFs and scanned image-based documents.

The foundational prompt for this workflow (see [[summaries/extract_pdf]]) captures the essential use case:
- Target a **folder of PDFs** as the input corpus
- Extract **numbers**, **dates**, and **key information**
- Deliver results in **spreadsheet format**

## Extraction Targets

| Data Type | Examples |
|---|---|
| Numbers | Scores, totals, measurements, IDs |
| Dates | Report dates, test dates, deadlines |
| Key Information | Names, diagnoses, conclusions, labels |

## Techniques and Tools

### OCR-Based Extraction
Scanned PDFs require Optical Character Recognition before any text parsing can occur. See [[concepts/ocr-pipeline]] for details on how image-based documents are converted to machine-readable text.

### Structured PDF Parsing
Native digital PDFs can be parsed directly using libraries that preserve layout and positional data. [[concepts/docling-pdf-parsing]] describes one such approach used in this knowledge base.

### Score-Specific Extraction
In clinical and neuropsychological contexts, extraction is often focused on test scores and norms. See [[concepts/pdf-score-extraction]] for domain-specific pipelines.

### Vector Search and Retrieval
Once extracted, text chunks can be embedded and retrieved semantically. [[concepts/retrieval-augmented-generation]] and [[concepts/vector-search]] describe how extracted data feeds downstream AI workflows.

## Output Formats

- **Spreadsheet (CSV/XLSX)**: Row-per-document or row-per-data-point formats
- **Long-format tables**: Ideal for multi-document batch processing (see [[concepts/long-format-clinical-data]])
- **Parquet files**: Efficient columnar storage for large corpora (see [[concepts/parquet-as-knowledge-store]])
- **DuckDB staging**: SQL-queryable extracted data (see [[concepts/duckdb-data-staging]])

## Clinical Applications

In neuropsychological and clinical workflows, PDF extraction is a critical upstream step:
- Parsing assessment PDFs to feed score interpretation (see [[concepts/neuropsychological-test-scores]])
- Extracting data for automated narrative generation (see [[concepts/narrative-report-generation]])
- Pulling structured results from clinical PDF assessments (see [[concepts/clinical-pdf-assessment]])
- Supporting the broader neuropsychological assessment pipeline (see [[concepts/neuropsychological-assessment-pipeline]])

## Challenges

- **Layout variability**: Different PDF templates require different parsing logic
- **Scanned vs. digital**: OCR introduces error rates not present in native PDFs
- **Table detection**: Multi-column tables are notoriously difficult to parse accurately
- **PHI sensitivity**: Clinical PDFs contain protected health information requiring careful handling (see [[concepts/phi-data-handling]] and [[concepts/pii-redaction-pipelines]])

## Related Concepts

- [[concepts/ocr-pipeline]]
- [[concepts/docling-pdf-parsing]]
- [[concepts/pdf-score-extraction]]
- [[concepts/text-chunking]]
- [[concepts/rag-chunking]]
- [[concepts/clinical-data-management]]
- [[concepts/duckdb-data-staging]]
- [[concepts/parquet-as-knowledge-store]]
- [[concepts/neuropsychological-assessment-pipeline]]
- [[concepts/phi-data-handling]]


See also: [[summaries/DIAGNOSIS_FIX_SUMMARY]]