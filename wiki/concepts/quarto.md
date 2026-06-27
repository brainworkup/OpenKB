---
sources: [summaries/Apply-to-Y-Combinator-JWT.md, summaries/copilot-instructions.md, summaries/LLM_INTEGRATION.md, summaries/LLM_AGENT_MAP.md, summaries/CLAUDE.md, summaries/2026-04-26-cingulate-agent-team-design.md, summaries/DEPENDENCIES.md, summaries/0007-voice-modular-report-sections-via-quarto-includes.md, summaries/style-training-to-report-drafting.md, summaries/report-rendering-pipeline.md, summaries/brand-yml-integration.md, summaries/report-template.md, summaries/brand-and-skills.md, summaries/0010-voice-quarto-typst-reporting.md, summaries/0007-style-modular-report-sections-via-quarto-includes.md, summaries/0006-brand-yml-for-cross-platform-theming.md, summaries/0001-voice-record-architecture-decisions.md, summaries/README.md, summaries/RECOVERY_NOTES.md, summaries/neuropsych-narrative-writer.md, summaries/responses_to_claude.md, summaries/issue_branding_typst.md, summaries/report-generation.md, summaries/template-system.md, summaries/quarto-extensions.md, summaries/overview.md, summaries/003-modular-template-structure.md, summaries/001-choose-quarto-typst.md, summaries/quarto.md]
brief: Quarto is the publishing layer for modular, branded clinical report pipelines.
---

# Quarto Publishing System

Quarto is an open-source scientific and technical publishing system that produces HTML documents, websites, RevealJS presentations, dashboards, and PDF files from a unified authoring format (`.qmd` files). It integrates deeply with R, Python, and other languages, and supports rich theming and branding through [[concepts/brand-theming]] via the `_brand.yml` file. In this knowledge base, Quarto is a core orchestration layer for professional document pipelines, especially neuropsychological reporting, where executable code, structured templates, reusable sections, and publication-quality PDF output are essential.

The canonical architecture decision for this project (ADR-0010, see [[summaries/0010-voice-quarto-typst-reporting]]) confirms Quarto as the orchestration layer and [[concepts/typst-typesetting]] as the primary PDF rendering engine, consolidating two prior overlapping ADRs into a single authoritative record. The cingulate R package (see [[summaries/README]]) builds directly on this foundation, using Quarto and Typst templates under `inst/quarto/` and `inst/rmarkdown/` as its report output layer. The newer Luria framing described in [[summaries/Apply-to-Y-Combinator-JWT]] reinforces the same role: Quarto sits near the end of a local-first, agent-based neuropsychological workflow, where upstream agents collect, organize, interpret, and synthesize clinical data before rendering comprehensive reports.

## Core Capabilities

- **Multi-format output**: A single source file can render to HTML, PDF (via [[concepts/typst-typesetting]] or LaTeX), RevealJS slides, or dashboard formats
- **Project system**: A `_quarto.yml` file at the project root configures output formats, navigation, and shared settings for all pages
- **Automatic brand discovery**: Quarto ≥1.4 detects `_brand.yml` at the project root and applies it without explicit configuration — it can also be explicitly referenced as `brand: brand/_brand.yml` in `_quarto.yml`
- **Format-level brand override**: Individual documents can override the project-level brand in their frontmatter (e.g., `format: html: brand: _brand.yml`)
- **Language integration**: Deep support for R, Python, and Julia via [[concepts/r-python-integration]], with executable code chunks
- **YAML-driven configuration**: All document and project settings are expressed through [[concepts/yaml-configuration]] in frontmatter or `_quarto.yml`
- **Template system**: Supports reusable Quarto extensions, variable substitution (e.g., `_variables.yml`), and modular section includes for parameterized report generation
- **Include mechanism**: The `{{< include file.qmd >}}` shortcode enables modular assembly of complex documents from discrete section files, with each section file self-contained with its own R setup chunks and Typst formatting
- **Extension system**: Custom format extensions can bundle templates, styles, fonts, and variables for domain-specific report types — see [[concepts/quarto-extensions]]
- **Dynamic inclusion**: A dispatcher file (e.g., `_domains_to_include.qmd`) can conditionally include sections based on available data, enabling reports to adapt automatically to administered assessments

## Integration with the cingulate R Package

The cingulate R package ([[summaries/README]]) uses Quarto as a core output layer for [[concepts/neuropsychological-assessment-automation]]. Its report templates live under:

- `inst/quarto/` — Quarto/Typst report templates and `_extensions` (e.g., neurotyp adult/pediatric/forensic)
- `inst/rmarkdown/` — R Markdown skeleton templates (pluck PDFs, drilldown)

The package's high-level API generates Quarto driver files:

```r
create_patient_workspace("ExamplePatient", age = 12)
results <- process_all_domains("data", age_group = "child")
generate_assessment_report(
  results,
  patient_info = list(name = "ExamplePatient", age = 12)
)
```

This means Quarto is not just a standalone tool but is orchestrated programmatically by the [[concepts/r6-class-architecture]] within cingulate, enabling fully automated report drafting from raw test scores through to publication-quality PDFs. LLM-assisted narrative content is injected into Quarto include files such as `_02-05_memory_text.qmd`, which are then assembled into the final report.

In the broader Luria system described in [[summaries/Apply-to-Y-Combinator-JWT]], Quarto fits naturally as the final publishing layer after upstream clinical processing. The founder describes a three-year evolution from R and RMarkdown toward LaTeX, then Quarto, and now Typst-backed report production. In that framing, Quarto serves as the bridge between structured clinical data processing and final neuropsychological report delivery, supporting a workflow that is local-first, privacy-sensitive, and increasingly agent-driven. This makes Quarto especially valuable in systems where report generation is not an isolated formatting task but the final synthesis stage of a larger [[concepts/neuropsychological-assessment-workflow]].

## Brand and Skills Integration

Quarto participates in a broader skill-based development system. The `brand-yml` skill module (at `skills/brand-yml/SKILL.md`) provides decision trees and reference documentation for creating, modifying, and troubleshooting `_brand.yml` files across Quarto, Shiny R, and Shiny Python. A dedicated `quarto` skill module covers Quarto authoring patterns and alt-text best practices via sub-skills `quarto-alt-text` and `quarto-authoring`. These [[concepts/skills-modules]] make Quarto theming and authoring patterns reusable and AI-assistant-friendly.

The `_brand.yml` specification defines a visual identity contract consumed simultaneously by multiple rendering targets:

| Target | Integration Method |
|---|---|
| **Quarto** | Auto-discovered at project root, or explicit `brand: brand/_brand.yml` in `_quarto.yml` |
| **Shiny (R)** | `bslib::bs_theme(brand = TRUE)` — auto-discovers when `_brand.yml` is at app root |
| **Shiny (Python)** | `ui.Theme.from_brand(__file__)` — requires `pip install "shiny[theme]"` |
| **Typst PDF** | Indirect — Quarto translates brand colors/typography to Typst variables during rendering |

**Current state**: The `brand/` directory exists but is empty — no `_brand.yml` has been created yet. The `_quarto.yml` file references `brand: brand/_brand.yml` and will fail until the file is created. A `brand-yml.prompt` at project root contains the specification prompt for AI-assisted brand.yml generation.

### Recommended brand.yml Creation Workflow

1. Gather brand info (colors, fonts, logos, company info)
2. Read `references/brand-yml-spec.md` for the full specification
3. Build incrementally: colors → typography → logos → meta
4. Validate: YAML syntax, color references, font definitions, file paths
5. Test across targets: Quarto HTML, Typst PDF, Shiny

See [[summaries/brand-and-skills]] for the full skills and brand system reference, [[summaries/brand-yml-spec]] for the `_brand.yml` format specification, and [[summaries/brand-yml-integration]] for the complete integration map.

## Brand and Theming Integration

Quarto reads `_brand.yml` and maps its sections to format-specific styling systems:

- **Color**: Exposed as Sass variables (`$brand-{color-name}`) in HTML/SCSS and as `brand-color.{name}` variables in Typst templates. See [[concepts/brand-color-system]].
- **Typography**: Applied to base text, headings, monospace, and links. See [[concepts/brand-typography]].
- **Light/Dark mode**: Any color or typography value can have `light:` and `dark:` variants for sites with theme toggles.

### Typst PDF Branding

Brand values are translated to Typst variables during rendering:

- `color.primary` → Typst `fill` values for headings/links
- `typography.base` → Typst `set text(font: ...)`
- `typography.headings` → Typst `show heading: set text(font: ...)`

**Note**: Not all brand.yml features have Typst equivalents. The Typst templates in `style/_extensions/brainworkup/` define their own font/paper defaults that may override brand.yml values depending on precedence.

### Theme Layering

Theme layering uses a priority list where the `brand` keyword controls where `_brand.yml` sits relative to other themes:

```yaml
format:
  html:
    theme:
      - cosmo      # lowest priority
      - brand      # middle
      - custom.scss # highest priority
```

See [[summaries/quarto]] for full theme layering examples.

### Accessing Brand Values in Documents

- **SCSS**: `$brand-primary`, `$brand-background`
- **Typst**: `brand-color.primary`, `brand-background-color.{name}`
- **Shortcodes**: Require extensions (e.g., `{< brand-color primary >}`)

## Report Generation Use Case

Quarto is well-suited for professional document pipelines. In the Voice Style project (see [[summaries/001-choose-quarto-typst]] and [[summaries/0010-voice-quarto-typst-reporting]]), Quarto was selected alongside [[concepts/typst-typesetting]] as the foundation for generating neuropsychological reporting outputs across multiple report types (pediatric, adult, forensic). The same logic is reinforced by the Luria product narrative in [[summaries/Apply-to-Y-Combinator-JWT]], where report writing is described as both the most time-consuming and the most important deliverable in neuropsychological evaluation. Quarto is a strong fit for that environment because it supports reproducible, modular, computation-backed report generation while preserving domain-specific structure and clinical voice.

Key reasons for this choice:

- Native R code chunk execution via `knitr` for data visualization
- YAML-based configuration enabling reproducible builds
- Strong integration with the R ecosystem
- Support for Quarto extensions that bundle templates, styles, and variables
- Better maintainability and version control compared to Word templates or LaTeX
- **Faster compilation**: Typst is typically much faster than LaTeX's multi-pass compilation
- **Simpler templating**: Typst's rule/function model is more approachable than LaTeX's macro-heavy approach
- **Font simplicity**: Typst uses system fonts directly, reducing configuration overhead
- **Clinical modularity**: Sections can be composed, revised, and regenerated independently across evaluation types
- **Privacy-compatible deployment**: Quarto works well in local-first pipelines where sensitive clinical data should remain on device or within tightly controlled infrastructure

**Alternatives considered and rejected** for PDF rendering:
- **LaTeX**: Powerful but steep learning curve, slow multi-pass compilation, complex error messages
- **Pandoc alone**: Less structured, harder to maintain complex templates
- **Word templates**: Not reproducible, no code execution, poor version control

The primary trade-off of choosing Typst over LaTeX is ecosystem maturity: Typst has a younger package ecosystem, and missing features may require workarounds or upstream contributions.

See [[summaries/001-choose-quarto-typst]] and [[summaries/0010-voice-quarto-typst-reporting]] for the full architecture decision records.

## Report Type Extensions

The Voice Style system demonstrates Quarto's extension mechanism for clinical report generation. Three custom format extensions live under `style/_extensions/brainworkup/`, each targeting a different clinical audience:

| Extension | Audience | Font | Paper Size | Heading Font |
|---|---|---|---|---|
| `neurotyp-pediatric` | Children | Equity B | A4 | Source Sans 3 |
| `neurotyp-adult` | Adults | IBM Plex Serif | US Letter | IBM Plex Sans |
| `neurotyp-forensic` | Forensic | TeX Gyre Termes | US Letter | IBM Plex Sans |

All three use APA citation style, 11.5pt (forensic: 12pt) font size, and do not number sections. Each extension provides:
- `typst-template.typ` — page layout, geometry, margins, headers/footers, and font definitions
- `typst-show.typ` — styling show rules for headings, paragraphs, lists, tables, figures, and code blocks
- `_extension.yml` — format configuration and metadata (title, author, version, quarto-required)

The `_extension.yml` structure registers template partials:

```yaml
contributes:
  formats:
    typst:
      template-partials:
        - typst-template.typ
        - typst-show.typ
```

New report types can be added by creating a new extension directory, defining `_extension.yml`, creating the Typst files, and registering the format in the project's `_quarto.yml`. Extensions follow semantic versioning (MAJOR.MINOR.PATCH).

See [[summaries/quarto-extensions]] for the full extension reference and [[concepts/quarto-extensions]] for cross-document synthesis.

## Modular Template Architecture

For complex reports such as neuropsychological evaluations, Quarto's include mechanism enables a **modular template system** where a main `template.qmd` orchestrates discrete section files. This approach was formalized in ADR 003 (see [[summaries/003-modular-template-structure]]) and further refined in two complementary ADR 0007 records — one for the Style project (see [[summaries/0007-style-modular-report-sections-via-quarto-includes]]) and one for the Voice project (see [[summaries/0007-voice-modular-report-sections-via-quarto-includes]]) — both arriving at the same design. It is a key pattern in the [[concepts/modular-report-architecture]] concept.

The neuropsychological report is composed of many semi-independent sections corresponding to distinct cognitive and clinical domains: test scores, behavioral observations, memory findings, executive function, ADHD, emotion, diagnoses, recommendations, signature, and appendix. These sections vary per patient and evaluation type, making flexible composition essential. This modularity also aligns with the Luria product direction, where different agents or processing stages can prepare domain-specific content that is later assembled into a unified report.

### Directory Structure

The full template hierarchy under `inst/quarto/templates/typst-report/` (cingulate/Voice project) looks like this:

```text
typst-report/
├── template.qmd                    # Main orchestrator
├── _quarto.yml                     # Template configuration
├── _variables.yml                  # Template variables
├── config.yml                      # Template config
├── _domains_to_include.qmd        # Dynamic dispatcher
├── _00-00_tests.qmd               # Tests section
├── _01-00_nse.qmd                 # NSE section
├── _01-01_behav_obs.qmd           # Behavioral observations
├── _02-05_memory.qmd              # Memory domain
├── _02-06_executive.qmd           # Executive function
├── _02-09_adhd.qmd                # ADHD assessment
├── _02-10_emotion.qmd             # Emotion/psychopathology
├── _03-00_dsm5_icd10_dx.qmd       # Diagnoses
├── _03-00_sirf.qmd                # SIRF summary
├── _03-00_sirf_text.qmd           # SIRF text
├── _03-01_recs.qmd                # Recommendations
├── _03-02_signature.qmd           # Report signature
├── _03-03_appendix.qmd            # Appendix
├── _03-03a_informed_consent.qmd   # Informed consent
└── _03-03b_examiner_qualifications.qmd  # Qualifications
```

### Numbered Prefix System

Section files use a two-part numbering prefix (`XX-YY_sectionname.qmd`) that encodes hierarchy and ordering. This naming convention was explicitly adopted in ADR 0007 to enforce ordering in the file system and clarify section hierarchy:

| Prefix | Section | Description |
|--------|---------|-------------|
| `00-XX` | Tests | Assessment battery and test list |
| `01-00` | NSE | Neuropsychological Status Exam |
| `01-01` | Behav Obs | Behavioral observations |
| `02-XX` | Domains | Cognitive domain assessments |
| `03-00` | Diagnoses / SIRF | DSM-5/ICD-10 diagnoses; Summary, Impairments, Recommendations, Findings |
| `03-01` | Recs | Recommendations |
| `03-02` | Signature | Report signature |
| `03-03` | Appendix | Appendix and consent forms |

### Main Template Setup

The `template.qmd` YAML frontmatter carries patient-level metadata:

```yaml
---
title: NEUROCOGNITIVE EXAMINATION
patient: Biggie
name: Smalls, Biggie
doe: "YYYY-MM-DD"
date_of_report: last-modified
---
```

The R setup chunk configures the execution environment, loading packages, establishing paths, and setting rendering options. In more advanced pipelines, it can also configure local model backends, caching behavior, and typography defaults before downstream sections are included.

### Include Mechanism

Sections are assembled via Quarto's include shortcode:

```quarto
{{< include _01-00_nse.qmd >}}
{{< include _domains_to_include.qmd >}}
```

Each section file is self-contained with its own R setup chunks and Typst formatting, enabling:
- **Section reuse**: Individual sections (e.g., `_02-09_adhd.qmd`) can be included or excluded per report without modifying shared template logic
- **Parallel editing**: Multiple contributors can edit different sections simultaneously with minimal merge conflicts
- **Dynamic inclusion**: `_domains_to_include.qmd` acts as a dispatcher, including domain sections based on available data
- **Agent-compatible assembly**: Upstream processing components can populate or revise individual section inputs without needing to rewrite the entire document

### Variables System

Shared variables are defined in `_variables.yml` and consumed across all sections:

```yaml
patient: Biggie
first_name: Biggie
last_name: Smalls
age: 18
dob: "XXXX-XX-XX"
doe: "2025-01-01"
case_number: "CASE-001"
```

These variables are accessible in three contexts:
- **Quarto markdown**: `{{< var patient >}}`
- **Typst blocks**: `#let patient = [{{< var patient >}}]`
- **R code**: `Sys.getenv("PATIENT")`

The configuration hierarchy uses three files:
- **`config.yml`**: Patient info, data paths, processing options, model settings
- **`_quarto.yml`**: Render settings, format definitions, execution options, figure settings
- **`_variables.yml`**: Patient demographics, report metadata, dynamic content

See [[concepts/neuropsychological-report-variables]] for the cross-document synthesis of variable handling across the neuropsychological reporting pipeline.

### Benefits of Modular Design

- **Reusability**: Same section file used across pediatric, adult, and forensic report variants
- **Maintainability**: Changes to one section are isolated and don't affect others
- **Flexibility**: Add or remove sections without modifying the main template
- **Collaboration**: Multiple team members can work on different sections simultaneously
- **Testability**: Individual sections can be validated independently
- **Clinical synthesis support**: Domain-level outputs can be prepared upstream and integrated into one final report shell

**Trade-offs**: More files to manage; all contributors must understand the include mechanism; variable naming must remain consistent across sections; each section's self-contained setup may lead to some duplication across files; the dispatcher logic must be maintained as new sections are added.

## Data Flow in Clinical Pipelines

In both the Voice Style system and the cingulate package, Quarto sits at the center of a multi-stage pipeline:

```text
Raw PDF → LLM Extraction (Python/local models) → Structured Data (JSON/CSV)
  → DuckDB Staging → Domain Processors (R6)
  → Quarto Template (template.qmd) → R/knitr Execution
  → Typst Rendering → Final PDF Report
```

The cingulate package adds a [[concepts/duckdb-data-staging]] layer upstream of Quarto and uses model-generated domain summaries to produce narrative text that is injected into Quarto include files. Downstream, [[concepts/typst-typesetting]] renders the final PDF. R packages execute within Quarto's knitr engine to produce tables, visualizations, and processed scores. See [[summaries/overview]] for the full system architecture.

The Luria product narrative in [[summaries/Apply-to-Y-Combinator-JWT]] adds an important product-level interpretation of this flow. There, Quarto is part of a larger local-first clinical automation stack intended to handle sensitive neuropsychological evaluations from data collection through report production. In that architecture, Quarto is valuable because it converts structured analysis into polished clinical deliverables while remaining compatible with [[concepts/local-first-architecture]], [[concepts/phi-data-handling]], and [[concepts/multi-agent-orchestration]].

## Supported Output Formats

| Format | Brand Support |
|---|---|
| HTML documents | Full |
| HTML dashboards | Full |
| RevealJS presentations | Full |
| Typst PDFs | Full (with font caching) |
| Multi-page websites | Full |

## Typst PDF Output

Quarto integrates with [[concepts/typst-typesetting]] for PDF generation. Brand colors and typography are passed into Typst templates. Google Fonts are automatically downloaded and cached at `.quarto/typst-font-cache/`. Font issues can be debugged with:

```bash
quarto typst fonts --ignore-system-fonts --font-path .quarto/typst-font-cache/
```

For structured report pipelines, Typst templates can be implemented per report type and configured through the Quarto project file. Each report type extension defines its own `typst-template.typ` and `typst-show.typ` for independent layout and style control.

In clinical settings, this Quarto→Typst handoff is especially useful because it combines reproducible computation with fast PDF compilation and tightly controlled formatting. That makes it practical for high-volume, iteration-heavy report workflows such as those described in [[summaries/Apply-to-Y-Combinator-JWT]].

## Brand Extensions

Branding can be packaged as reusable Quarto extensions, bundling `brand.yml`, logos, and fonts:

```bash
quarto create extension brand
quarto add username/my-brand-extension
```

Extensions declare their contribution via `contributes.brand` in `_extension.yml` and require a `_quarto.yml` project file to activate. The same extension mechanism supports domain-specific report templates (e.g., `style/_extensions/brainworkup/` in the Voice project and `inst/quarto/` in the cingulate package).

## Troubleshooting

- **Brand not applied**: Ensure `brand: brand/_brand.yml` is set in `_quarto.yml` or that `_brand.yml` is present at the project root; note that the `brand/` directory is currently empty and must be populated first
- **Include errors**: Verify file path, check for syntax errors in the included file, review Quarto render log
- **Variable substitution failures**: Confirm variable name in `_variables.yml` matches exactly; check variable scoping
- **R execution issues**: Check knitr options in setup chunk, verify required packages are installed, review cache settings
- **Font issues**: Use `quarto typst fonts` to debug font availability in the cache; Typst requires system fonts to be installed
- **Dispatcher not including a section**: Verify the data availability condition in `_domains_to_include.qmd` and confirm the section file's numeric prefix is correct
- **Missing Typst packages**: Typst's younger ecosystem may lack packages available in LaTeX; workarounds or upstream contributions may be needed
- **Privacy-sensitive deployment issues**: In local-first clinical pipelines, verify that temporary files, caches, and rendered artifacts are stored in approved locations and handled consistently with [[concepts/clinical-data-privacy]]
- **Shiny integration failures**: For R, confirm `bslib` is installed and `bs_theme(brand = TRUE)` is used; for Python, confirm `shiny[theme]` is installed and `_brand.yml` is at app root

## Related Pages

- [[summaries/README]] — cingulate R package README: LLM super agent for neuropsychological assessment reporting
- [[summaries/001-choose-quarto-typst]] — ADR selecting Quarto + Typst for neuropsychological report generation
- [[summaries/0010-voice-quarto-typst-reporting]] — Consolidated canonical ADR confirming Quarto + Typst and Typst-vs-LaTeX rationale
- [[summaries/003-modular-template-structure]] — ADR defining the modular section include system
- [[summaries/0007-style-modular-report-sections-via-quarto-includes]] — ADR formalizing modular section files and naming conventions (Style project)
- [[summaries/0007-voice-modular-report-sections-via-quarto-includes]] — ADR formalizing modular section files and naming conventions (Voice project)
- [[summaries/0006-brand-yml-for-cross-platform-theming]] — ADR establishing brand.yml as the cross-platform theming contract
- [[summaries/brand-yml-integration]] — Integration map for brand.yml across Quarto, Shiny R, and Shiny Python
- [[summaries/template-system]] — Full reference for the modular QMD template system
- [[summaries/overview]] — Component overview of the Voice Style system
- [[summaries/quarto]] — Full guide to using `_brand.yml` with Quarto
- [[summaries/quarto-extensions]] — Reference for the three neuropsychological report format extensions
- [[summaries/brand-yml-spec]] — The brand definition file format specification
- [[summaries/brand-yml-in-r]] — Brand theming in the R ecosystem
- [[summaries/brand-and-skills]] — Brand system and skills modules reference
- [[summaries/Apply-to-Y-Combinator-JWT]] — YC application draft describing Luria as a local-first, agent-based neuropsych workflow
- [[concepts/quarto-extensions]] — Cross-document synthesis of the Quarto extension system
- [[concepts/typst-typesetting]] — The Typst typesetting engine used for PDF output
- [[concepts/brand-theming]] — Cross-document synthesis of brand theming approaches
- [[concepts/brand-color-system]] — Color palette and semantic color mapping
- [[concepts/brand-typography]] — Typography configuration across tools
- [[concepts/yaml-configuration]] — YAML as a configuration language
- [[concepts/r-python-integration]] — Quarto's polyglot execution model
- [[concepts/r-visualization-theming]] — Theming plots and visuals in R/Python within Quarto
- [[concepts/modular-report-architecture]] — Design patterns for modular document assembly
- [[concepts/cognitive-domains]] — Cognitive domain framework assessed in neuropsychological reports
- [[concepts/neuropsychological-report-variables]] — Variable handling in neuropsychological reporting pipelines
- [[concepts/neuropsychological-reporting]] — Cross-document synthesis of neuropsychological report structure
- [[concepts/skills-modules]] — Reusable AI-assisted development knowledge modules
- [[concepts/r6-class-architecture]] — R6 class-based modular processing architecture
- [[concepts/duckdb-data-staging]] — DuckDB-backed data staging for neuropsychological test data
- [[concepts/neuropsychological-assessment-automation]] — Automated pipeline from raw scores to polished reports
- [[concepts/neuropsychological-assessment-workflow]] — End-to-end neuropsychological evaluation workflow
- [[concepts/local-first-architecture]] — Local-first system design for sensitive workflows
- [[concepts/phi-data-handling]] — Handling protected health information in clinical systems
- [[concepts/multi-agent-orchestration]] — Multi-agent coordination in complex pipelines
- [[concepts/clinical-data-privacy]] — Privacy and data protection in clinical software

See also: [[summaries/report-generation]] | [[summaries/issue_branding_typst]] | [[summaries/responses_to_claude]] | [[summaries/neuropsych-narrative-writer]] | [[summaries/RECOVERY_NOTES]] | [[summaries/0001-voice-record-architecture-decisions]] | [[summaries/report-template]]

See also: [[summaries/report-rendering-pipeline]]

See also: [[summaries/style-training-to-report-drafting]]

See also: [[summaries/DEPENDENCIES]]

See also: [[summaries/2026-04-26-cingulate-agent-team-design]]

See also: [[summaries/CLAUDE]]

See also: [[summaries/LLM_AGENT_MAP]]

See also: [[summaries/LLM_INTEGRATION]]

See also: [[summaries/copilot-instructions]]