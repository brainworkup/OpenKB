---
doc_type: short
full_text: sources/template-system.md
---

# Template System

## Overview

The template system composes neuropsychological reports from modular Quarto Markdown (QMD) files, enabling reusability, maintainability, and flexibility across different report types. It integrates [[concepts/quarto]], [[concepts/typst-typesetting]], and the neuro2 package to render structured clinical documents.

## Directory Structure

All template files reside under `style/templates/typst-report/` and include:

- `template.qmd` — Main orchestrator that includes all sections
- `_quarto.yml` — Template-specific Quarto configuration
- `_variables.yml` — Patient and case variables
- `config.yml` — Processing and patient defaults
- Numbered section files (e.g., `_01-00_nse.qmd`, `_02-05_memory.qmd`)

## Numbering Convention

Section files use a `XX-YY_sectionname.qmd` prefix scheme:

| Prefix | Category |
|--------|----------|
| 00 | Header / test battery |
| 01 | Interview / behavioral observations |
| 02 | Cognitive domain assessments |
| 03 | Conclusions, diagnoses, appendix |

## Main Template Mechanics

- **YAML frontmatter** defines patient name, date of exam, and report date.
- **R setup chunk** loads `dplyr`, `readr`, `here`, `yaml`, and `neuro2`, calling `neuro2::setup_neuro2()` to configure LLM backend (Ollama), caching, fonts, and figure defaults.
- **Typst header block** maps Quarto variables to Typst `#let` bindings for use in layout.
- **Section includes** use `{{< include _file.qmd >}}` directives; domain sections can be conditionally included via `_domains_to_include.qmd`.

## Variables System

Defined in `_variables.yml` (patient name, DOB, DOE, case number, age), variables are consumed in multiple contexts — see [[concepts/neuropsychological-report-variables]] and [[concepts/yaml-configuration]]:

- In Quarto markdown: `{{< var patient >}}`
- In Typst: `#let patient = [{{< var patient >}}]`
- In R: `Sys.getenv("PATIENT")`

## Key Sections

- **_00-00_tests.qmd**: Test battery list with versions, dates, scores — related to [[concepts/neuropsychological-tests]].
- **_01-00_nse.qmd**: Neuropsychological Status Exam — mental status, effort/validity indicators.
- **_01-01_behav_obs.qmd**: Behavioral presentation, affect, speech, motor, attention.
- **_02-XX domain files**: Memory, executive function, ADHD, emotion — each with results, interpretation, and normative comparisons; see [[concepts/cognitive-domains]] and [[concepts/neuropsychological-test-scores]].
- **_03-00_dsm5_icd10_dx.qmd**: DSM-5/ICD-10 diagnoses and rule-outs.
- **_03-00_sirf.qmd / _sirf_text.qmd**: Summary of Impairments, Recommendations, and Findings (SIRF), a structured summary format; see [[concepts/clinical-report-structure]].
- **_03-01_recs.qmd**: Treatment, accommodation, referral, and follow-up recommendations.
- **_03-02_signature.qmd / _03-03_appendix.qmd**: Credentials, informed consent, examiner qualifications.

## Configuration

`_quarto.yml` sets project type, render target (`template.qmd`), and links `_variables.yml` as a metadata file. `config.yml` mirrors global config for patient defaults and processing options (DuckDB, parallel execution). See [[concepts/quarto-extensions]] for further Quarto configuration patterns.

## Creating and Extending Sections

1. Create a new QMD with the appropriate numeric prefix.
2. Add content and R code chunks using shared variables.
3. Register the file in `template.qmd`'s include list.
4. Test independently before integration.

## Troubleshooting

- **Include failures**: verify file path and syntax.
- **Variable substitution failures**: confirm exact name match in `_variables.yml`.
- **R execution failures**: check knitr options, package installation, and cache settings.

## Related Concepts

- [[concepts/quarto]]
- [[concepts/typst-typesetting]]
- [[concepts/modular-report-architecture]]
- [[concepts/neuropsychological-reporting]]
- [[concepts/neuropsychological-report-variables]]
- [[concepts/neuropsychological-assessment-pipeline]]
- [[concepts/yaml-configuration]]
- [[concepts/r-python-integration]]