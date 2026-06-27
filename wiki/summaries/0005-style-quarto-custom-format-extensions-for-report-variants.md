---
doc_type: short
full_text: sources/0005-style-quarto-custom-format-extensions-for-report-variants.md
---

# Summary: Quarto Custom Format Extensions for Report Variants

**Source:** 0005-style-quarto-custom-format-extensions-for-report-variants
**Date:** 2025-01-20
**Status:** Accepted

## Overview

This [[concepts/architecture-decision-records|Architecture Decision Record (ADR)]] documents the choice to implement **three separate Quarto custom format extensions** for neuropsychological report variants, rather than a single monolithic template or one extension with conditional logic.

## Problem Context

Neuropsychological reports exist in three variants:
- **Pediatric** — targets child patients
- **Adult** — targets adult patients
- **Forensic** — targets legal/forensic contexts

All three share approximately 80% of their [[concepts/typst-typesetting]] layout logic but differ in:
- **Fonts**: Equity B (pediatric) vs. Libertinus Serif/Sans (adult, forensic)
- **Margins and paper size**: US Letter (pediatric, adult) vs. A4 (forensic)
- **Heading styles** and list/link styling (forensic has enhanced styling)
- **Font sizes**: 11.5pt (pediatric) vs. 11pt (adult, forensic)

## Decision

Three separate [[concepts/quarto-extensions]] under `style/_extensions/brainworkup/`:

| Extension | Font | Paper | Size |
|---|---|---|---|
| `neurotyp-pediatric` | Equity B | US Letter | 11.5pt |
| `neurotyp-adult` | Libertinus Serif | US Letter | 11pt |
| `neurotyp-forensic` | Libertinus Serif/Sans | A4 | 11pt |

Each extension provides:
- `_extension.yml` — [[concepts/yaml-configuration|Quarto extension metadata]]
- `typst-template.typ` — Typst document template
- `typst-show.typ` — Typst show rules

Users select a variant in `_quarto.yml` via `format: neurotyp-pediatric-typst` (or `-adult`, `-forensic`).

## Consequences

### Benefits
- **Variant isolation**: Changes to one report type cannot accidentally break another.
- **Clear selection**: Simple format key in `_quarto.yml` selects the full variant stack.

### Trade-offs
- **Duplicated logic**: Common patterns (header rendering, run-in subheadings, confidential watermark) are repeated across three templates.
- **Maintenance surface**: Three extensions require three updates for any shared change (e.g., logo path, header format).
- **Future refactor opportunity**: Shared Typst logic could be extracted into reusable [[concepts/typst-modules]] to reduce duplication.

## Related Concepts
- [[concepts/clinical-report-structure]]
- [[concepts/architecture-decision-records]]
- [[concepts/quarto-extensions]]
- [[concepts/quarto]]
- [[concepts/typst-typesetting]]
- [[concepts/typst-modules]]
- [[concepts/modular-report-architecture]]
- [[concepts/forensic-neuropsychological-evaluation]]
- [[concepts/neuropsychological-reporting]]