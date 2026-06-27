---
doc_type: short
full_text: sources/overview.md
---

# Component Overview — Voice Style System

## Overview

The **Voice Style** system is a modular neuropsychological report generation framework built on [[concepts/quarto]] and [[concepts/typst-typesetting]]. It orchestrates the full pipeline from raw PDF test data to polished clinical reports, supporting pediatric, adult, and forensic report types.

## System Architecture

The top-level layout:

- `style/_extensions/brainworkup/` — Custom Quarto format extensions
- `style/templates/typst-report/` — Modular Quarto/Typst report templates
- `config.yml` / `_quarto.yml` / `_variables.yml` — Configuration layers
- `soul/` — Python processing and AI/LLM scripts

## Core Components

### 1. Quarto Extensions

Three custom formats under `style/_extensions/brainworkup/`:

| Extension | Audience | Font | Paper |
|---|---|---|---|
| `neurotyp-pediatric` | Children | Equity B | A4 |
| `neurotyp-adult` | Adults | IBM Plex Serif | US Letter |
| `neurotyp-forensic` | Forensic | TeX Gyre Termes | US Letter |

Each extension provides a `typst-template.typ` and `typst-show.typ` for consistent [[concepts/typst-typesetting]] rendering. See also [[concepts/quarto-extensions]] for background on the extension mechanism.

### 2. Template System

Modular section files with a numbered prefix system inside `style/templates/typst-report/`:

| Prefix | Section |
|---|---|
| 00-00 | Tests / assessment battery |
| 01-00 | Neuropsychological Status Exam (NSE) |
| 01-01 | Behavioral Observations |
| 02-XX | Cognitive Domain Assessments |
| 03-00 | Diagnoses (DSM-5/ICD-10) & SIRF |
| 03-01 | Recommendations |
| 03-02 | Signature |
| 03-03 | Appendix / Consent |

The `template.qmd` acts as the main orchestrator, including all section files. This pattern is described in detail in [[concepts/modular-report-architecture]] and [[concepts/clinical-report-structure]].

### 3. Configuration System

- **`config.yml`**: Patient info, data paths, processing options, MCP/LLM settings
- **`_quarto.yml`**: Render settings, format definitions, execution options, figure settings
- **`_variables.yml`**: Patient demographics, report metadata, dynamic content

Configuration management across these layered files is an instance of [[concepts/yaml-configuration]].

### 4. Processing Scripts (`soul/`)

Python scripts for data processing and AI operations:

- `extract_pdf_data.py` / `extract_pdf_data_enhanced.py` — PDF data extraction
- `neuro_report_style_agent.py` — Report style AI agent
- `main.py` — Main entry point

These scripts form part of the broader [[concepts/neuropsychological-assessment-pipeline]] and draw on [[concepts/clinical-pdf-assessment]] and [[concepts/pdf-score-extraction]] techniques.

## Data Flow

```
Raw PDF (test reports)
    → MCP LLM Extraction (soul/)
    → Structured Data (JSON/CSV)
    → Quarto Template (template.qmd)
    → R Code Execution (knitr)
    → Typst Rendering
    → Final PDF Report
```

## Integration Points

- **[[concepts/model-context-protocol]] (Ollama)**: PDF extraction, clinical interpretation, lookup table integration
- **[[concepts/local-llm-inference]]**: Local LLM backend powering the MCP integration
- **R / neuro2 package**: Neuropsychological data processing, ggplot2 visualization, dplyr, knitr — see [[concepts/r-python-integration]]
- **[[concepts/typst-typesetting]]**: Custom templates, show rules, font management, page layout

## Extension Points

- **New report types**: Add extension folder, define `_extension.yml`, create Typst files
- **New sections**: Create prefixed QMD, add to orchestrator, use shared variables
- **Custom processing**: Modify `soul/` scripts, update MCP config, add tools or lookup tables

## Key Concepts

- [[concepts/quarto]] — Document rendering and format management
- [[concepts/quarto-extensions]] — Custom format extension mechanism
- [[concepts/typst-typesetting]] — Type-setting and PDF generation
- [[concepts/model-context-protocol]] — AI/LLM integration for clinical data extraction
- [[concepts/neuropsychological-reporting]] — Domain context for report structure and content
- [[concepts/modular-report-architecture]] — Section-based template design pattern
- [[concepts/clinical-report-structure]] — Clinical document organization conventions
- [[concepts/python-project-structure]] — Layout of the `soul/` processing scripts

## Related Concepts
- [[concepts/clinical-data-privacy]]
