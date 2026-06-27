---
sources: [summaries/README.md, summaries/CLAUDE.md, summaries/agent-team.md, summaries/0007-voice-modular-report-sections-via-quarto-includes.md, summaries/report-rendering-pipeline.md, summaries/brand-yml-integration.md, summaries/style-extensions.md, summaries/report-template.md, summaries/0010-voice-quarto-typst-reporting.md, summaries/0007-style-modular-report-sections-via-quarto-includes.md, summaries/0005-style-quarto-custom-format-extensions-for-report-variants.md, summaries/neuropsych-narrative-writer.md, summaries/SKILL.md, summaries/overview.md, summaries/report-generation.md, summaries/template-system.md, summaries/quarto-extensions.md]
brief: Quarto extensions package Typst templates and format metadata into reusable units for neuropsychological report variants.
---

# Quarto Extensions for Custom Formats

Quarto extensions allow authors to package [[concepts/typst-typesetting]] templates, show rules, and format metadata into self-contained, reusable units that can be shared and versioned independently from the documents that use them. In the context of neuropsychological reporting, extensions define the full visual and structural presentation layer for each report type.

See [[summaries/quarto-extensions]], [[summaries/0005-style-quarto-custom-format-extensions-for-report-variants]], [[summaries/style-extensions]], and [[summaries/overview]] for source document details. The rendering pipeline that invokes these extensions is documented in [[summaries/report-rendering-pipeline]].

## What a Quarto Extension Is

A Quarto extension is a directory registered under `_extensions/` that contributes one or more output formats. For Typst-based formats, an extension bundles:

- **`_extension.yml`** — manifest declaring title, author, version, minimum Quarto version, and which template files are contributed
- **`typst-template.typ`** — page geometry, margins, fonts, headers, footers, and global document settings; defines the `#let report(...)` function
- **`typst-show.typ`** — glues Quarto YAML variables to the `report()` function via `$if$` conditionals, controlling heading styles, paragraph spacing, list formatting, table styling, figure captions, and block quotes

This design reflects a clean separation between layout logic (`typst-template.typ`) and data binding (`typst-show.typ`). Once installed, the extension's format name (e.g., `neurotyp-pediatric-typst`) becomes available in any project's `_quarto.yml`.

## Extension Configuration

The `_extension.yml` file follows this pattern:

```yaml
contributes:
  formats:
    typst:
      template-partials:
        - typst-template.typ
        - typst-show.typ
```

This wires the Typst partials into Quarto's render pipeline. Projects can then override specific settings (papersize, font, fontsize) in their own `_quarto.yml` without modifying the extension itself.

Extension versioning uses semantic versioning (MAJOR.MINOR.PATCH), enabling downstream projects to pin to stable releases.

In `_quarto.yml`, usage looks like:

```yaml
format:
  neurotyp-pediatric-typst:   # or neurotyp-adult-typst / neurotyp-forensic-typst
    keep-typ: true
    keep-md: true
    fontsize: 11.5pt
```

The `keep-typ` and `keep-md` flags preserve intermediate files useful for debugging rendering output.

## Selecting a Format in the Rendering Pipeline

The format key set in `_quarto.yml` is the primary control point for which extension is invoked during `quarto render`. The full rendering pipeline proceeds as follows once a format is selected:

1. Quarto reads `_quarto.yml` and `_variables.yml`
2. The format extension (e.g., `neurotyp-pediatric`) is resolved from `style/_extensions/brainworkup/`
3. R chunks are executed via knitr (data processing, figure generation)
4. `{{< include >}}` shortcodes assemble the full document from section partials
5. Quarto generates intermediate Typst source (`.typ`)
6. Typst compiles the source to PDF

Patient-specific variables (name, DOB, age, sex, diagnoses, pronouns) are injected into both QMD content and Typst blocks via `{{< var key >}}` shortcodes sourced from `_variables.yml`, making the same extension reusable across patients. See [[concepts/neuropsychological-report-variables]] for the full variable schema.

## The Three brainworkup Extensions

An explicit architecture decision (ADR-0005, accepted 2025-01-20; see [[concepts/architecture-decision-records]]) established that **three separate Quarto custom format extensions** — rather than a single monolithic template or one extension with conditional logic — are the correct approach for neuropsychological report variants. The rationale: the three variants share roughly 80% of layout logic but differ enough in fonts, margins, paper size, and heading styles to warrant isolation.

All three extensions live under `style/_extensions/brainworkup/`, each tuned for a distinct clinical context:

### neurotyp-pediatric
- **Format key**: `neurotyp-pediatric-typst`
- **Body/Heading font**: Equity B (bold)
- **Paper**: US Letter, 11.5pt
- **Margins**: top 30mm, right 25mm, bottom 30.25mm, left 25mm
- **Version**: 0.1.9999 (pre-release)

Targets patients under 18. Designed for developmental norms, educational implications, and family-focused recommendations.

### neurotyp-adult
- **Format key**: `neurotyp-adult-typst`
- **Body/Heading font**: Libertinus Serif (bold)
- **Paper**: US Letter, 11pt
- **Margins**: 1.25in × 1.25in
- **Version**: 0.1.3

Targets adult patients (18+). Suited for work-related assessments, disability evaluations, and cognitive aging.

### neurotyp-forensic
- **Format key**: `neurotyp-forensic-typst`
- **Body font**: Libertinus Serif; **Heading font**: Libertinus Sans (semibold)
- **Paper**: A4, 11pt
- **Margins**: 25mm × 30mm
- **Extras**: Customized list spacing, link styling, optimized line breaks
- **Version**: 0.1.0

Targets legal and forensic evaluations. Emphasizes methodology documentation and legal standards adherence — aligning with the register requirements described in [[concepts/forensic-neuropsychological-evaluation]].

| Extension | Font | Paper | Size | Version |
|---|---|---|---|---|
| `neurotyp-pediatric` | Equity B | US Letter | 11.5pt | 0.1.9999 |
| `neurotyp-adult` | Libertinus Serif | US Letter | 11pt | 0.1.3 |
| `neurotyp-forensic` | Libertinus Serif/Sans | A4 | 11pt | 0.1.0 |

All three share APA citation style and do not number sections.

## Common Typst Patterns

All three templates share a set of design conventions:

- **Confidential header**: Shown on pages >1 with patient name and date in smallcaps
- **Run-in subheadings**: Level 4+ headings render as italic inline labels followed by a colon
- **Logo block**: Top of page 1 renders `inst/resources/logo.png` (or `img/logo.png` for adult/forensic)
- **Centered title**: `NEUROCOGNITIVE EXAMINATION` in 1.75em bold
- **Table styling**: `inset: 6pt`, `stroke: none`
- **Link styling**: Dark fill `rgb(4, 1, 23)`, weight 450, underlined

These shared conventions ensure visual consistency across [[concepts/neuropsychological-reporting]] regardless of clinical context.

## Figure Font Matching

A subtle but important detail in the rendering pipeline is font consistency between figures and body text. The R setup chunk in `template.qmd` calls a `pick_font()` function that detects available system fonts and configures `svglite` and `ggplot2` to use the same typeface as the active Typst extension. This means SVG figures rendered by R will visually match the document's body font — Equity B for pediatric, Libertinus Serif for adult and forensic reports. System font availability is therefore a prerequisite for correct rendering.

## Consequences of the Three-Extension Design

### Benefits
- **Variant isolation**: Changes to one report type cannot accidentally break another.
- **Clear selection**: Users specify the variant in `_quarto.yml` via `format: neurotyp-pediatric-typst` (or `-adult`, `-forensic`), which applies the full visual and structural stack.

### Trade-offs
- **Duplicated logic**: Common patterns — header rendering, run-in subheadings, confidential watermark, logo block — are repeated across all three templates.
- **Maintenance surface**: Three extensions require three updates for any shared change such as a logo path or header format.
- **Future refactor opportunity**: Shared Typst logic could be extracted into reusable Typst modules to reduce duplication without sacrificing variant isolation.

## Role in the Neuropsych Pipeline

Within the [[concepts/luria-neuropsych-pipeline]], Quarto extensions serve as the presentation destination for prose generated upstream. Specifically:

- **Stage 3** of the pipeline (the narrative writer agent described in [[summaries/neuropsych-narrative-writer]]) produces per-domain prose as Quarto include files (`_NN-XX_<domain>_text.qmd`) that target these extensions.
- The R6 / cingulate layer renders score tables and plots and assembles the final Typst PDF using the extension's show rules and template.
- The narrative writer targets one of the three extensions (`neurotyp-adult`, `neurotyp-pediatric`, or `neurotyp-forensic`) depending on the `age_group` field in the extracted data.

This means that extensions are not just visual wrappers — they are the **contractual output target** for the entire multi-stage pipeline. The narrative writer is explicitly prohibited from producing any output format other than `.qmd` files destined for this Quarto+Typst rendering path.

## Role in the Voice Style System

Within the broader Voice Style framework, extensions are one layer of a multi-layered architecture:

- Extensions handle **presentation** (fonts, paper size, layout, show rules)
- Content partials (`_00-00_tests.qmd`, `_01-00_nse.qmd`, etc.) handle **content**
- `_variables.yml` and `config.yml` handle **data and configuration**
- The `template.qmd` orchestrator assembles all layers into a final document

This separation allows, for example, a pediatric report and an adult report to share identical section content while rendering with completely different visual treatments. The [[concepts/modular-report-architecture]] concept governs how these layers interact.

## Section Inclusion Chain

The `template.qmd` file assembles reports via `{{< include >}}` shortcodes in a fixed order. The full chain for a standard report is:

```text
_00-00_tests.qmd
_01-00_nse.qmd
_01-01_behav_obs.qmd
[Typst heading: NEUROCOGNITIVE FINDINGS]
_domains_to_include.qmd  ← dynamic dispatcher
[pagebreak]
_03-00_sirf.qmd
_03-00_sirf_text.qmd
_03-01_recs.qmd
_03-02_signature.qmd
[pagebreak]
_03-03_appendix.qmd
```

This ordering is stable and must be preserved, as both `_quarto.yml` and the extension template depend on it.

## Per-Domain Include File Naming Convention

The narrative writer agent writes files following a stable prefix scheme that extensions and `_quarto.yml` depend on:

| Prefix | Domain |
|---|---|
| `_02-01_iq` | General Cognitive Ability / IQ |
| `_02-05_memory` | Memory & Learning |
| `_02-06_executive` | Executive Function |
| `_02-09_adhd` | ADHD (multi-rater variants) |
| `_03-00_sirf` | Summary, Impressions, Recommendations & Formulation |

These prefixes must never be renumbered, as both `_quarto.yml` and the extension's template depend on the stable ordering. See [[summaries/neuropsych-narrative-writer]] for the full domain table.

## Relationship to the Broader Stack

- **[[concepts/quarto]]** — The host system that discovers, loads, and invokes extensions during document rendering
- **[[concepts/typst-typesetting]]** — The underlying composition engine whose template language populates the extension files
- **[[concepts/modular-report-architecture]]** — Extensions are one layer of modularity; they handle presentation while content partials handle data
- **[[concepts/yaml-configuration]]** — Both `_extension.yml` and `_quarto.yml` use YAML as the configuration language
- **[[concepts/neuropsychological-reporting]]** — The clinical domain that motivates the three distinct format variants
- **[[concepts/r-python-integration]]** — R (via knitr) and Python scripts supply processed data that flows through the template before Typst renders the final PDF
- **[[concepts/narrative-report-generation]]** — The prose content that the narrative writer agent produces and that extensions ultimately render
- **[[concepts/luria-neuropsych-pipeline]]** — The multi-stage pipeline for which these extensions serve as the terminal rendering target
- **[[concepts/architecture-decision-records]]** — ADR-0005 formally captures the rationale for three separate extensions over alternatives
- **[[concepts/forensic-neuropsychological-evaluation]]** — The clinical and legal context driving the distinct forensic extension variant
- **[[concepts/neuropsychological-report-variables]]** — The patient variable schema injected via `_variables.yml` into extension templates

## Creating a New Extension

1. Create a directory: `style/_extensions/brainworkup/<new-type>/`
2. Write `_extension.yml` with metadata and format contributions
3. Write `typst-template.typ` for page structure (defining the `#let report(...)` function)
4. Write `typst-show.typ` for content styling rules (binding YAML variables via `$if$` conditionals)
5. Reference the new format name in the project's `_quarto.yml`
6. Validate with a test render

## Troubleshooting

| Symptom | Likely Cause |
|---|---|
| Extension not found | Wrong path in `_quarto.yml` or malformed `_extension.yml` |
| Typst compilation error | Syntax error in `.typ` files or missing font |
| Format not applied | Format name mismatch or extension not loaded |
| Narrative include not rendered | Domain prefix mismatch between `.qmd` filename and `_quarto.yml` listing |
| Figure font mismatch | Required system font (Equity B, Libertinus) not installed |

Checking the Quarto render output and Typst error messages are the primary diagnostic tools.

See also: [[summaries/report-rendering-pipeline]]

See also: [[summaries/template-system]]

See also: [[summaries/report-generation]]

See also: [[summaries/overview]]

See also: [[summaries/SKILL]]

See also: [[summaries/neuropsych-narrative-writer]]

See also: [[summaries/style-extensions]]

See also: [[summaries/0005-style-quarto-custom-format-extensions-for-report-variants]]

See also: [[summaries/0007-style-modular-report-sections-via-quarto-includes]]

See also: [[summaries/0010-voice-quarto-typst-reporting]]

See also: [[summaries/report-template]]

See also: [[summaries/brand-yml-integration]]

See also: [[summaries/0007-voice-modular-report-sections-via-quarto-includes]]

See also: [[summaries/agent-team]]

See also: [[summaries/CLAUDE]]

See also: [[summaries/README]]