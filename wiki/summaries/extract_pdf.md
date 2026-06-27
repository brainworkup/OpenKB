---
doc_type: short
full_text: sources/extract_pdf.md
---

# Extract PDF

## Overview
This document is a short prompt or instruction template for using an AI or tool to extract structured data from a collection of PDF files located in a specified folder path.

## Core Request
The prompt instructs an agent to:
- Process **all PDFs** within a given folder
- Extract key data types:
  - **Numbers** (quantities, figures, measurements)
  - **Dates** (timestamps, deadlines, periods)
  - **Key information** (names, topics, summaries)
- Output the extracted data into a **spreadsheet format** (e.g., CSV or Excel)

## Use Cases
- Bulk document processing for data analysis
- Converting unstructured PDF reports into structured tables
- Automating data entry from scanned or digital documents

## Related Concepts
- [[concepts/clinical-pdf-assessment]]
- [[concepts/neuropsychological-assessment-pipeline]]
- [[concepts/pdf-data-extraction]] — Techniques for pulling structured data from unstructured PDF sources
- [[concepts/pdf-score-extraction]] — Extracting scored or numeric values from PDF documents
- [[concepts/ocr-pipeline]] — Optical character recognition workflows for handling scanned document collections
- [[concepts/docling-pdf-parsing]] — Parsing and processing PDF files for downstream use

## Notes
This is a minimal prompt template. In practice, additional detail would be needed such as the specific folder path, the target spreadsheet schema, and the output file format.