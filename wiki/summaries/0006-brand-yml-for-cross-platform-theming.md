---
doc_type: short
full_text: sources/0006-brand-yml-for-cross-platform-theming.md
---

# Summary: brand.yml for Cross-Platform Theming (ADR 0006)

**Date:** 2025-01-20
**Status:** Accepted

## Overview

This Architecture Decision Record (ADR) establishes `_brand.yml` (Posit's brand specification) as the single source of truth for visual identity across the project's output targets: Quarto documents, Shiny apps, and Typst PDFs.

## Problem Statement

The project required consistent visual identity — colors, typography, and logos — across multiple rendering targets. Three options were considered:

1. Duplicate theme configuration per target (rejected: maintenance burden)
2. A single source-of-truth theme file (chosen)
3. CSS/SCSS variables only (rejected: insufficient cross-platform reach)

## Decision

Adopt **`_brand.yml`** as the canonical brand specification file:

- **Location:** `brand/_brand.yml`
- **Quarto integration:** Referenced via `brand: brand/_brand.yml` in `_quarto.yml`
- **Skill support:** `skills/brand-yml/` provides specification docs and integration guides

## Key Consequences

### Benefits
- **Single source of truth:** Colors, fonts, and logos defined once, consumed by all targets
- **Auto-discovery:** [[concepts/quarto]] ≥1.4 and Shiny ≥1.9 automatically detect `_brand.yml` at project root
- **Skill-driven workflow:** `skills/brand-yml/SKILL.md` offers a decision tree for creating, modifying, and troubleshooting brand files

### Limitations
- **Typst limitation:** Integration with [[concepts/typst-typesetting]] is indirect — Quarto translates brand colors to Typst variables, but not all brand.yml features map cleanly

## Integration Points

| Target | Integration Method | Auto-discovery |
|---|---|---|
| [[concepts/quarto]] | `brand:` key in `_quarto.yml` | ≥ v1.4 |
| Shiny (R/Python) | Native brand.yml support | ≥ v1.9 |
| [[concepts/typst-typesetting]] | Quarto-mediated variable translation | Indirect |

## Related Concepts
- [[concepts/r-visualization-theming]]

- [[concepts/brand-theming]] — The broader cross-platform theming challenge this ADR addresses
- [[concepts/brand-color-system]] — Colors as a key brand asset managed by brand.yml
- [[concepts/brand-typography]] — Typography definitions within the brand specification
- [[concepts/yaml-configuration]] — The file format underlying the brand specification
- [[concepts/quarto]] — Primary document rendering target
- [[concepts/quarto-extensions]] — Extension mechanism relevant to Quarto brand integration
- [[concepts/architecture-decision-records]] — The ADR format used to document this decision
- [[concepts/r-python-integration]] — Relevant to Shiny's dual R/Python brand.yml support