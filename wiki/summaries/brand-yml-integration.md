---
doc_type: short
full_text: sources/brand-yml-integration.md
---

# brand.yml Integration

## Overview

This document describes how `_brand.yml` integrates with each rendering target in the voice project, covering [[concepts/quarto]] (HTML, Typst PDF, dashboards), Shiny for R, and Shiny for Python. The `brand/` directory currently exists but is empty — the file has not yet been created.

## Integration Targets

### Quarto

- Configured via `_quarto.yml` with `brand: brand/_brand.yml`
- Quarto ≥1.4 auto-discovers `_brand.yml` at project root
- Supports format-level overrides in document front matter
- Outputs: HTML, [[concepts/typst-typesetting]] (Typst PDF), Dashboards, RevealJS

#### Typst PDF Specifics

- `color.primary` → Typst `fill` for headings/links
- `typography.base` → Typst `set text(font: ...)`
- `typography.headings` → Typst `show heading: set text(...)`
- **Caveat**: Typst templates in `style/_extensions/brainworkup/` may override brand.yml depending on precedence; not all brand.yml features have Typst equivalents.

### Shiny for R

- Uses `bslib::bs_theme(brand = TRUE)` for auto-discovery when `_brand.yml` is at app root
- Explicit path also supported: `bs_theme(brand = "path/to/_brand.yml")`

### Shiny for Python

- Uses `ui.Theme.from_brand(__file__)` for auto-discovery
- Requires `pip install "shiny[theme]"`

## Current State

- `brand/` directory is **empty** — `_brand.yml` not yet created
- `_quarto.yml` references `brand: brand/_brand.yml` (will fail until file exists)
- Specification prompt available at `brand-yml.prompt` in project root
- Skill workflow documented in `skills/brand-yml/SKILL.md`

## Creating brand.yml — Recommended Workflow

1. Gather brand info: colors, fonts, logos, company info
2. Read `references/brand-yml-spec.md` for full specification
3. Build incrementally: colors → typography → logos → meta
4. Validate: YAML syntax, color references, font definitions, file paths
5. Test across all targets: Quarto HTML, Typst PDF, Shiny

## Related Concepts
- [[concepts/quarto-extensions]]
- [[concepts/skills-modules]]
- [[concepts/r-visualization-theming]]

- [[concepts/brand-theming]] — cross-platform brand theming via brand.yml
- [[concepts/brand-color-system]] — color specification and referencing
- [[concepts/brand-typography]] — font and typography configuration
- [[concepts/yaml-configuration]] — YAML syntax and structure
- [[concepts/typst-typesetting]] — Typst PDF rendering and variable translation
- [[concepts/quarto]] — Quarto project configuration and rendering
- [[concepts/typst-modules]] — Typst template extensions and overrides
- [[summaries/brand-yml-spec]] — full brand.yml specification reference
- [[summaries/brand-yml-in-r]] — Shiny for R brand.yml integration details