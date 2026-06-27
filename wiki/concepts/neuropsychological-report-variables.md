---
sources: [summaries/cerner-autotext.md, summaries/nt_interpretation.md, summaries/0007-voice-modular-report-sections-via-quarto-includes.md, summaries/customization.md, summaries/report-rendering-pipeline.md, summaries/style-extensions.md, summaries/report-template.md, summaries/0007-style-modular-report-sections-via-quarto-includes.md, summaries/SKILL.md, summaries/template-system.md]
brief: Centralized YAML-driven system for injecting patient-specific data across all layers of neuropsychological report generation.
---

# Neuropsychological Report Variables System

The neuropsychological report variables system is a structured approach to managing patient-specific data across all components of a clinical report. By centralizing variable definitions in a single YAML file, the system ensures consistency, reduces manual errors, and decouples patient data from report logic. This system works in close coordination with the Quarto custom format extensions (neurotyp-pediatric, neurotyp-adult, neurotyp-forensic) that determine how those variables are ultimately rendered in the final PDF.

## Core Mechanism

Variables are declared once in `_variables.yml` and consumed by every layer of the reporting stack:

```yaml
patient: Biggie
first_name: Biggie
last_name: Smalls
age: 18
dob: "XXXX-XX-XX"
doe: "2025-01-01"
case_number: "CASE-001"
dx1: ADHD
dx2: anxiety
sex: male
he_she: he
his_her: his
```

This single source of truth propagates into three different rendering contexts. The `_variables.yml` file can live at the project root or under `style/templates/typst-report/`, and is registered globally in `_quarto.yml` via `metadata-files`.

## Consumption Contexts

### Quarto Markdown
Variables are injected inline using Quarto's native variable syntax:
```markdown
{{< var patient >}}
{{< var doe >}}
```
This is used throughout narrative sections such as behavioral observations and recommendations.

### Typst Typesetting
Variables are bound to Typst `#let` declarations in the document header block:
```typst
#let name = [{{< var last_name >}}, {{< var first_name >}}]
#let doe = [{{< var date_of_report >}}]
#let case_number = [{{< var case_number >}}]
```
This enables precise layout control in the final rendered PDF. The neurotyp extensions expose these bindings through `typst-show.typ`, which maps Quarto YAML variables to the `report()` function defined in `typst-template.typ`. See [[concepts/typst-typesetting]] and [[concepts/typst-modules]] for details on how Typst handles document composition.

Variables can also be used directly inside raw Typst blocks within QMD files (`` ```{=typst} `` chunks), allowing patient data to appear in Typst-formatted header sections without requiring a separate template edit.

### R Code Chunks
R code within QMD sections can access patient context via environment variables or the `config.yml` file read through `yaml::read_yaml()`:
```r
patient <- Sys.getenv("PATIENT")
# or
cfg <- yaml::read_yaml("config.yml")
```
This allows dynamic data processing and score lookup to be scoped to the correct patient record. The `config.yml` file also controls LLM backend selection and data paths for the R processing pipeline, creating a parallel configuration path alongside `_variables.yml`.

## Pipeline Position

Within the full rendering pipeline, variable injection is the first step before any compilation occurs:

```text
_variables.yml ──┐
                 │
config.yml ──────┤
                 ├──► Quarto render ──► Typst compile ──► PDF
brand/_brand.yml─┤         │
                 │         ▼
_extensions/     ──  typst-template.typ
brainworkup/          typst-show.typ
```

Quarto reads `_variables.yml` at the start of the render pass, making all values available to QMD narrative, R chunks, and Typst blocks before Typst compilation begins. See [[summaries/report-rendering-pipeline]] for the complete step-by-step pipeline description.

## Integration with Style Extensions

The neurotyp format extensions (neurotyp-pediatric, neurotyp-adult, neurotyp-forensic) each implement a confidential header that displays the patient name and date of evaluation in smallcaps on pages after the first. This header is driven directly by the variables bound through `typst-show.typ`. Key variable-driven elements across all three extensions include:

- **Patient name**: Rendered in the confidential header via `#let name`
- **Date of evaluation**: Rendered alongside the name in the running header
- **Case/report metadata**: Used to populate the centered title block (`NEUROCOGNITIVE EXAMINATION`)
- **Diagnoses**: `dx1`, `dx2` fields used in narrative sections
- **Pronouns**: `he_she`, `his_her` fields enable grammatically correct narrative generation

The choice of extension variant (pediatric, adult, or forensic) affects typographic rendering and font selection but not the variable schema itself — the same `_variables.yml` definitions work across all three formats. See [[summaries/style-extensions]] for the full extension reference and [[summaries/0005-style-quarto-custom-format-extensions-for-report-variants]] for the architecture decision context.

## Integration with the YAML Configuration Layer

The variables system is linked into the project via `_quarto.yml`:
```yaml
metadata-files:
  - _variables.yml
```
And the active format extension is declared in the same file:
```yaml
format:
  neurotyp-pediatric-typst:
    keep-typ: true
    keep-md: true
    fontsize: 11.5pt
```
This makes the variables available globally across all included section files without repetition. See [[concepts/yaml-configuration]] for broader patterns in YAML-driven configuration.

## Relationship to Modular Template Architecture

Because variables are centralized, every modular section file (`_01-00_nse.qmd`, `_02-05_memory.qmd`, etc.) can reference patient data without duplicating it. This is fundamental to the [[concepts/modular-report-architecture]] pattern: sections remain generic and reusable, while the variables system provides the patient-specific context at render time.

The section inclusion chain in `template.qmd` assembles the full report from discrete files via `{{< include >}}` shortcodes, with patient variables flowing into each included file automatically through Quarto's global metadata scoping.

See [[summaries/template-system]] for the full directory structure and section naming conventions.

## Scope and Naming Conventions

- Variable names must match **exactly** between `_variables.yml` and their usage sites, including in `typst-show.typ` `$if$` conditionals.
- Quarto's variable scoping means variables defined in `_variables.yml` are accessible to all included files in the same project.
- The `config.yml` file mirrors some variables (e.g., patient name, age) for use by the R processing pipeline, creating a parallel configuration path.
- The neurotyp-adult and neurotyp-forensic extensions use `img/logo.png`, while neurotyp-pediatric uses `inst/resources/logo.png` — a path that may need to be set as a variable or build-time parameter.
- Font selection is handled separately: `pick_font()` in the R setup chunk detects system fonts and configures figure rendering to match document typography.

## Failure Modes and Troubleshooting

| Issue | Likely Cause |
|---|---|
| Variable renders as empty | Name mismatch between definition and usage |
| Typst binding fails | Quarto variable not resolved before Typst compilation |
| R environment variable missing | `setup_neuro2()` not called or `.Renviron` not configured |
| Confidential header missing patient name | `$if(name)$` conditional not satisfied in `typst-show.typ` |
| Wrong logo path | Extension variant uses different asset path convention |
| Figure font mismatch | `pick_font()` not finding expected system font |

## Related Concepts

- [[concepts/neuropsychological-reporting]] — The broader reporting workflow this system supports
- [[concepts/modular-report-architecture]] — How section modularity depends on shared variables
- [[concepts/yaml-configuration]] — YAML as a configuration and data-injection mechanism
- [[concepts/quarto]] — The rendering engine that consumes `_variables.yml`
- [[concepts/quarto-extensions]] — The extension mechanism that connects format variants to the variable system
- [[concepts/typst-typesetting]] — The typesetting layer that receives bound variable values
- [[concepts/typst-modules]] — Typst template and show file patterns used by neurotyp extensions
- [[concepts/neuropsychological-assessment-pipeline]] — The upstream pipeline that produces the data populating these variables
- [[concepts/phi-data-handling]] — Patient variables contain PHI and must be handled accordingly

See also: [[summaries/report-rendering-pipeline]]

See also: [[summaries/SKILL]]

See also: [[summaries/0007-style-modular-report-sections-via-quarto-includes]]

See also: [[summaries/report-template]]

See also: [[summaries/style-extensions]]

See also: [[summaries/customization]]

See also: [[summaries/0007-voice-modular-report-sections-via-quarto-includes]]

See also: [[summaries/nt_interpretation]]

See also: [[summaries/cerner-autotext]]