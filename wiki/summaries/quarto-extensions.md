---
doc_type: short
full_text: sources/quarto-extensions.md
---

# Quarto Extensions for Neuropsychological Reports

## Overview

Quarto extensions in the `style/_extensions/brainworkup/` directory provide custom format definitions for different neuropsychological report types. Each extension wraps [[concepts/typst-typesetting]] components into a reusable Quarto format that can be selected per-project. See also [[concepts/quarto-extensions]] for broader context on how extensions integrate with the [[concepts/quarto]] ecosystem.

## Extension Types

Three extensions are provided, each targeting a distinct clinical context:

| Extension | Use Case | Font | Paper | Font Size |
|---|---|---|---|---|
| `neurotyp-pediatric` | Patients under 18 | Equity B | A4 | 11.5pt |
| `neurotyp-adult` | Adults 18+ | IBM Plex Serif | US Letter | 11.5pt |
| `neurotyp-forensic` | Legal/forensic evals | TeX Gyre Termes | US Letter | 12pt |

All three use APA citation style, IBM Plex Sans (or Source Sans 3 for pediatric) as the heading font, and do not number sections.

## Extension File Structure

Each extension contains three files:

- **`_extension.yml`**: Declares metadata (title, author, version, quarto-required) and registers Typst template partials.
- **`typst-template.typ`**: Defines page geometry, margins, headers/footers, fonts, and section styling.
- **`typst-show.typ`**: Defines Typst show rules for headings, paragraph spacing, lists, tables, figures, block quotes, and code blocks.

## Configuration via `_extension.yml`

```yaml
contributes:
  formats:
    typst:
      template-partials:
        - typst-template.typ
        - typst-show.typ
```

Versioning follows semantic versioning (MAJOR.MINOR.PATCH). Current version is `0.1.9999` (development).

## Format Selection

In the project's [[concepts/yaml-configuration]] file (`_quarto.yml`), formats are referenced by name (e.g., `neurotyp-pediatric-typst`) and can be further customized with per-project overrides for papersize, font, heading-family, and fontsize.

## Clinical Use Cases

- **Pediatric**: Developmental norms, educational implications, family-focused recommendations.
- **Adult**: Work assessments, disability evaluations, cognitive aging, neurological conditions.
- **Forensic**: Legal standards, methodology documentation, disclaimer sections, expert witness preparation.

These report types connect directly to the broader [[concepts/neuropsychological-reporting]] workflow and the [[concepts/modular-report-architecture]] that structures content across report sections.

## Creating New Extensions

1. Create a new directory under `style/_extensions/brainworkup/`.
2. Add `_extension.yml`, `typst-template.typ`, and `typst-show.typ`.
3. Register the format in the project's `_quarto.yml`.
4. Test with sample data.

## Troubleshooting Notes

- Extension not found: verify path, check `_extension.yml` syntax, confirm Quarto version.
- Typst compile errors: check template syntax, verify font availability.
- Format not applied: confirm format name matches and extension is loaded.

## Related Concepts
- [[concepts/clinical-report-structure]]
- [[concepts/neuropsychological-assessment-pipeline]]

- [[concepts/typst-typesetting]] — Typst template and show rule authoring
- [[concepts/neuropsychological-reporting]] — Clinical distinctions between pediatric, adult, and forensic reports
- [[concepts/quarto]] — How extensions integrate with `_quarto.yml` and overall project layout
- [[concepts/modular-report-architecture]] — Structuring report content across format types
- [[concepts/yaml-configuration]] — Extension and format configuration via YAML