---
doc_type: short
full_text: sources/style-extensions.md
---

# Style Extensions (neurotyp)

Three Quarto custom format extensions located at `style/_extensions/brainworkup/` that render neuropsychological reports via [[concepts/typst-typesetting]]. Each extension targets a different clinical context (pediatric, adult, forensic) and shares a common structural pattern while differing in typographic defaults.

## Shared Structure

Every extension contains three core files:

| File | Purpose |
|---|---|
| `_extension.yml` | Declares format name, author, version, and `template-partials` |
| `typst-template.typ` | Defines the `#let report(...)` function — page setup, header, fonts, heading rules, logo, title block |
| `typst-show.typ` | Glues Quarto YAML variables to `report()` via `$if$` conditionals |

This pattern reflects a clean separation between layout logic (`typst-template.typ`) and data binding (`typst-show.typ`), consistent with [[concepts/quarto-extensions]] and [[concepts/typst-modules]].

## Extension Variants

### neurotyp-pediatric
- **Format key**: `neurotyp-pediatric-typst`
- **Body/Heading font**: Equity B
- **Paper**: US letter, 11.5pt
- **Margins**: top 30mm, right 25mm, bottom 30.25mm, left 25mm
- **Version**: 0.1.9999 (pre-release)

### neurotyp-adult
- **Format key**: `neurotyp-adult-typst`
- **Body/Heading font**: Libertinus Serif
- **Paper**: US letter, 11pt
- **Margins**: 1.25in × 1.25in
- **Version**: 0.1.3

### neurotyp-forensic
- **Format key**: `neurotyp-forensic-typst`
- **Body font**: Libertinus Serif
- **Heading font**: Libertinus Sans (semibold)
- **Paper**: A4, 11pt
- **Margins**: 25mm × 30mm
- **Extras**: Customized list spacing, link styling, optimized line breaks
- **Version**: 0.1.0

## Common Typst Patterns

All three templates share:

- **Confidential header**: Shown on pages >1, patient name and date in smallcaps
- **Run-in subheadings**: Level 4+ headings render as italic inline labels followed by a colon
- **Logo block**: Top of page 1 renders `inst/resources/logo.png` (or `img/logo.png` for adult/forensic)
- **Centered title**: `NEUROCOGNITIVE EXAMINATION` in 1.75em bold
- **Table styling**: `inset: 6pt`, `stroke: none`
- **Link styling**: Dark fill `rgb(4, 1, 23)`, weight 450, underlined

These shared conventions ensure visual consistency across [[concepts/neuropsychological-reporting]] regardless of clinical context, and relate to the broader [[concepts/clinical-report-structure]] used throughout the project.

## Usage

```yaml
format:
  neurotyp-pediatric-typst:   # or neurotyp-adult-typst / neurotyp-forensic-typst
    keep-typ: true
    keep-md: true
    fontsize: 11.5pt
```

Configured in the project's `_quarto.yml`. The `keep-typ` and `keep-md` flags preserve intermediate files useful for debugging [[concepts/typst-typesetting]] output.

## Key Relationships

- Connects to [[concepts/quarto-extensions]] for the extension mechanism
- Relates to [[concepts/typst-typesetting]] and [[concepts/typst-modules]] for the underlying PDF engine and template components
- Relevant to [[concepts/neuropsychological-reporting]] as the presentation layer for clinical documents
- Ties into [[concepts/forensic-neuropsychological-evaluation]] for the forensic variant's A4 paper and distinct typography
- Reflects [[concepts/brand-typography]] choices specific to each clinical audience
- See also [[summaries/0005-style-quarto-custom-format-extensions-for-report-variants]] for related architectural decisions

## Related Concepts
- [[concepts/neuropsychological-report-variables]]
- [[concepts/modular-report-architecture]]
- [[concepts/yaml-configuration]]
