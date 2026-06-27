---
doc_type: short
full_text: sources/report-rendering-pipeline.md
---

# Report Rendering Pipeline

Source: `report-rendering-pipeline`

Describes the full pipeline from patient variable configuration to final PDF neuropsychological report, using [[concepts/quarto]] as the document engine and [[concepts/typst-typesetting]] as the typesetting compiler.

## Pipeline Architecture

The pipeline assembles inputs from multiple configuration files and passes them through two compilation stages:

1. **Quarto render** — processes `.qmd` files, executes R chunks, resolves includes and shortcodes, produces intermediate Typst source.
2. **Typst compile** — converts the `.typ` intermediate into a final PDF.

Key input files:
- `_variables.yml` — patient-specific metadata
- `_quarto.yml` / `config.yml` — format selection and R/LLM configuration
- `brand/_brand.yml` — visual branding
- `_extensions/brainworkup/` — format-specific Typst templates (`typst-template.typ`, `typst-show.typ`)

## Patient Variable Injection

Patient data (name, DOB, age, sex, diagnoses, pronouns) is defined in `_variables.yml` and injected via `{{< var key >}}` shortcodes into:
- QMD narrative content
- R chunk contexts
- Raw Typst blocks (`` ```{=typst} `` chunks)

This makes the same template reusable across patients by editing a single [[concepts/yaml-configuration]] file, and is the core mechanism behind [[concepts/neuropsychological-report-variables]].

## Report Format Selection

Three format profiles are available, selected in `_quarto.yml`:
- `neurotyp-pediatric-typst` — uses Equity B font
- `neurotyp-adult-typst` — uses Libertinus Serif/Sans
- `neurotyp-forensic-typst` — uses Libertinus Serif/Sans

Each maps to a corresponding [[concepts/quarto-extensions]] under `style/_extensions/brainworkup/`, and the overall approach reflects [[concepts/modular-report-architecture]].

## Section Inclusion Chain

`template.qmd` assembles the report via `{{< include >}}` shortcodes in a fixed order:
- Tests administered (`_00-00_tests.qmd`)
- Neurological status exam and behavioral observations
- Neurocognitive findings (dispatched dynamically via `_domains_to_include.qmd`)
- Summary/impressions, recommendations, signature, appendix

This inclusion chain is also documented in [[summaries/0007-style-modular-report-sections-via-quarto-includes]].

## Figure Font Matching

The R setup chunk calls `pick_font()` to detect available system fonts and configures `svglite` + `ggplot2` so SVG figures use the same typeface as the Typst document body, ensuring visual consistency. This integrates with [[concepts/r-visualization-theming]] and [[concepts/brand-typography]].

## Data Flow Table

| Source | Mechanism | Target |
|---|---|---|
| `_variables.yml` | `{{< var key >}}` | QMD content, R chunks |
| `_variables.yml` | `{{< var key >}}` in Typst blocks | Typst header |
| `_quarto.yml` format opts | Extension system | `typst-show.typ` conditionals |
| `config.yml` | `yaml::read_yaml()` in R | LLM backend, data paths |

## Prerequisites

- **Quarto** ≥1.4.0
- **Typst** (bundled or standalone)
- **R packages**: `neuro2`, `dplyr`, `readr`, `here`, `yaml`, `ggplot2`, `svglite`
- **System fonts**: Equity B (pediatric), Libertinus Serif/Sans (adult/forensic)

## Related Concepts
- [[concepts/narrative-report-generation]]
- [[concepts/neuropsychological-reporting]]
- [[concepts/clinical-report-structure]]
- [[concepts/brand-theming]]

- [[concepts/quarto]] — document engine driving the pipeline
- [[concepts/typst-typesetting]] — typesetting engine producing the PDF
- [[concepts/neuropsychological-report-variables]] — mechanism for patient data parameterization
- [[concepts/modular-report-architecture]] — format profiles and extension structure
- [[concepts/quarto-extensions]] — format-specific extension bundles
- [[concepts/neuropsychological-assessment-pipeline]] — broader clinical pipeline context
- [[concepts/r-python-integration]] — R execution within the Quarto render step
- [[concepts/typst-modules]] — Typst template components (`typst-template.typ`, `typst-show.typ`)