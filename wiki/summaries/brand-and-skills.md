---
doc_type: short
full_text: sources/brand-and-skills.md
---

# Brand & Skills

## Overview

This document describes two major infrastructure components for Posit/Quarto-based projects: the **brand system** (visual identity via `_brand.yml`) and **skills** (reusable AI-assisted development knowledge modules).

## Brand System

The brand system lives in `brand/` and is driven by Posit's `_brand.yml` specification. It centralizes visual identity — colors, typography, logos, and metadata — and is consumed by multiple rendering targets.

### `_brand.yml` Structure

| Section | Purpose |
|---|---|
| `meta` | Company name, links |
| `logo` | Image paths, light/dark variants |
| `color` | Palette + semantic colors (primary, success, warning, etc.) |
| `typography` | Fonts (base, headings, monospace), sizes, weights |
| `defaults` | Bootstrap/Shiny/Quarto-specific overrides |

### Integration Points

- **[[concepts/quarto]]**: Auto-discovered via `brand: brand/_brand.yml` in `_quarto.yml`
- **Shiny (R)**: `bs_theme(brand = TRUE)` via `bslib`
- **Shiny (Python)**: `ui.Theme.from_brand(__file__)`
- **Typst**: Indirect — Quarto translates brand colors to Typst variables at render time

The specification prompt is at `brand-yml.prompt`; the skill definition is at `skills/brand-yml/SKILL.md`.

## Skills

Skills are reusable knowledge modules stored in `skills/`, each with a `SKILL.md` or `README.md`. They contain decision trees, reference documentation, and integration patterns to guide AI-assisted development. See [[concepts/skills-modules]] for the broader pattern.

### Available Skills

#### `brand-yml`
- **Purpose**: Create, modify, and troubleshoot `_brand.yml` files
- **Decision tree flow**: Creating → Shiny R → Shiny Python → Quarto → Modifying → Troubleshooting
- **References**: spec, shiny-r, shiny-python, quarto, brand-yml-in-r docs

#### `quarto`
- **Purpose**: Quarto authoring and alt-text
- **Sub-skills**: `quarto-alt-text`, `quarto-authoring`

#### `posit-dev`
- **Purpose**: Posit development patterns
- **Sub-skills**: `critical-code-reviewer`, `describe-design`

#### `shiny`
- **Purpose**: Shiny app development with bslib theming
- **Sub-skills**: `shiny-bslib`, `shiny-bslib-theming`

#### `tidyverse`
- **Purpose**: Tidyverse R coding patterns

## Key Concepts

- [[concepts/brand-theming]] — The `_brand.yml` specification as a cross-tool visual identity contract
- [[concepts/brand-color-system]] — Palette and semantic color definitions
- [[concepts/brand-typography]] — Font and typographic configuration across platforms
- [[concepts/quarto]] — Quarto rendering and brand integration
- [[concepts/skills-modules]] — AI-assisted development knowledge modules pattern
- [[concepts/yaml-configuration]] — YAML-based configuration patterns used by `_brand.yml`
- [[concepts/r-python-integration]] — Cross-language Shiny theming via bslib

## Related Concepts
- [[concepts/r-visualization-theming]]
- [[concepts/ide-ai-assistant-configuration]]
