---
doc_type: short
full_text: sources/report-generation.md
---

# Report Generation Workflow

## Overview

This document describes the multi-stage pipeline for transforming raw psychological test data (PDFs) into professionally formatted neuropsychological reports using the Voice Style system. The workflow integrates local LLM inference, R statistical processing, [[concepts/quarto]] document rendering, and [[concepts/typst-typesetting]].

## System Requirements

- **Quarto** >=1.4.0
- **R** >=4.0 (with neuro2, dplyr, ggplot2, knitr, etc.)
- **Typst** (latest)
- **Ollama** (local LLM backend)
- **Python** >=3.13 (processing scripts)

Configuration is managed through `config.yml`, `_variables.yml`, and a selected Quarto extension (pediatric/adult/forensic). See [[concepts/yaml-configuration]] for configuration patterns.

## Pipeline Stages

### Stage 1: Data Preparation
Raw PDF test reports are placed in `data/raw/pdf/`. Paths and a clinical lookup table are registered in `config.yml`.

### Stage 2: PDF Data Extraction
A Python script (`soul/extract_pdf_data.py`) invokes an [[concepts/model-context-protocol]] server which uses [[concepts/local-llm-inference]] via Ollama (e.g., `llama3.1`) to parse PDFs, extract structured test scores, apply clinical terminology from the lookup table, and save results as JSON to `results/`. This stage is a key part of the [[concepts/neuropsychological-assessment-pipeline]].

### Stage 3: Data Processing (R/neuro2)
Structured JSON is loaded into R. The neuro2 package applies age-appropriate norms, calculates [[concepts/neuropsychological-test-scores]], generates visualizations (ggplot2), and creates summary tables. Parallel processing and DuckDB integration are supported. See [[concepts/r-python-integration]] for details on mixing R and Python in the pipeline.

### Stage 4: Template Rendering (Quarto)
[[concepts/quarto]] loads `template.qmd`, executes R setup chunks, substitutes variables from `_variables.yml`, includes ordered section files, and produces an intermediate Typst markup file (`.typ`). This step relies on the [[concepts/modular-report-architecture]] defined by the template system.

### Stage 5: Typst Compilation
The [[concepts/typst-typesetting]] compiler reads the `.typ` file, applies the selected extension template (neurotyp-pediatric, neurotyp-adult, neurotyp-forensic), and outputs the final PDF report. Extension selection is described further in [[concepts/quarto-extensions]].

## Command-Line Execution

```bash
# Full pipeline
cd style/templates/typst-report
quarto render template.qmd

# Extract PDF data only
python soul/extract_pdf_data.py

# Render specific format
quarto render template.qmd --to neurotyp-pediatric-typst
quarto render template.qmd --to neurotyp-adult-typst
quarto render template.qmd --to neurotyp-forensic-typst
```

## Output Locations

| Artifact | Location |
|---|---|
| Final PDF Report | `output/` |
| Intermediate Quarto cache | `.quarto/` |
| Structured JSON data | `results/` |
| Plots & Tables | Inline in report |

## Performance Optimization

- **R chunk caching**: `knitr::opts_chunk$set(cache = TRUE, autodep = TRUE)`
- **Parallel processing**: `processing: parallel: yes` in `config.yml`
- **Skip existing assets**: `options(neuro2.skip_if_exists = TRUE)`

## Quality Assurance

**Pre-generation**: Verify PDF completeness, patient info accuracy, test battery match, and lookup table entries.

**Post-generation**: Review PDF formatting, section inclusion, table/figure rendering, diagnostic codes, signature block, and recommendations. These checks align with the [[concepts/clinical-report-structure]] standards for neuropsychological reporting.

## Troubleshooting

- **PDF extraction fails**: Verify Ollama is running (`ollama list`), check PDF path and MCP server logs.
- **R code errors**: Verify package installation, R version, clear Quarto cache.
- **Typst errors**: Check installation, extension files, font availability.
- **Missing variables**: Verify `_variables.yml` completeness and YAML syntax.

## Key Integration Points

- [[concepts/model-context-protocol]]: PDF extraction, clinical interpretation, NLP
- [[concepts/neuropsychological-assessment-pipeline]]: Core processing workflow
- [[concepts/local-llm-inference]]: Local LLM inference backend via Ollama
- [[concepts/quarto]]: Document generation framework
- [[concepts/typst-typesetting]]: Final typesetting and PDF compilation
- [[concepts/phi-data-handling]]: Patient data privacy considerations throughout the pipeline

## Related Concepts
- [[concepts/neuropsychological-reporting]]
- [[concepts/clinical-nlp-pipelines]]
- [[concepts/clinical-data-privacy]]
