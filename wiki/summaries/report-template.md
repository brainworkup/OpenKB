---
doc_type: short
full_text: sources/report-template.md
---

# Report Template (typst-report)

## Overview

`style/templates/typst-report/` is the master [[concepts/quarto]] project that assembles a complete [[concepts/neuropsychological-reporting]] document from discrete, modular `.qmd` section files. It targets multiple output formats via [[concepts/typst-typesetting]] and is tightly coupled with the `neuro2` internal R package.

## File Structure

The template is split into numbered section files that map to clinical report sections:

| Range | Purpose |
|---|---|
| `_00-xx` | Test score tables |
| `_01-xx` | NSE screen, behavioral observations |
| `_02-xx` | Domain findings (memory, executive, ADHD, emotion) |
| `_03-xx` | Diagnoses, SIRF, recommendations, signature, appendix |

A special file, `_domains_to_include.qmd`, acts as a **dynamic dispatcher** that conditionally includes only the relevant cognitive domain sections for each patient.

## Master Document Flow

`template.qmd` orchestrates assembly in a fixed order:

1. **YAML front matter** — patient name, exam date, report date
2. **R setup chunk** — loads `neuro2`, sets [[concepts/local-llm-inference]] backend (Ollama), configures fonts for SVG output
3. **Typst header block** — structured patient metadata (case number, DOB, exam dates)
4. **Include chain** — sequential inclusion of section files, with a dynamic domain block and page breaks at logical boundaries

## Configuration Layer

Three config files drive the pipeline:

- **`_quarto.yml`** — defines four report format targets (pediatric, adult, forensic, vanilla) with per-format font and paper settings; see [[concepts/yaml-configuration]]
- **`_variables.yml`** — patient demographics, clinician info, pronouns, diagnoses; injected via `{{< var key >}}` shortcodes throughout the document; see [[concepts/neuropsychological-report-variables]]
- **`config.yml`** — pipeline flags for data I/O, report format selection, and MCP/LLM endpoint settings

## R Runtime Dependencies

| Package | Role |
|---|---|
| `neuro2` | Core internal package; setup, data processing, LLM integration |
| `dplyr`, `readr`, `here`, `yaml` | Data wrangling |
| `ggplot2`, `svglite` | Figure rendering with document-matched fonts |

## Key Design Patterns

- **Modularity** — each clinical section is an isolated `.qmd` file, enabling selective inclusion per case; see [[concepts/modular-report-architecture]]
- **Multi-format output** — a single source tree produces pediatric, adult, forensic, and generic variants via [[concepts/typst-modules]] format targets
- **LLM-assisted narrative** — the `neuro2` setup chunk configures an Ollama backend, suggesting [[concepts/narrative-report-generation]] is used for generating or refining prose sections
- **Variable injection** — patient-specific data flows through `_variables.yml` shortcodes, keeping clinical content separate from template logic

## Related Concepts
- [[concepts/quarto-extensions]]
- [[concepts/llm-provider-abstraction]]
- [[concepts/r-python-integration]]

- [[concepts/neuropsychological-reporting]]
- [[concepts/modular-report-architecture]]
- [[concepts/quarto]]
- [[concepts/typst-typesetting]]
- [[concepts/neuropsychological-report-variables]]
- [[concepts/narrative-report-generation]]
- [[concepts/local-llm-inference]]
- [[concepts/clinical-report-structure]]