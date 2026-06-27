---
doc_type: short
full_text: sources/quarto.md
---

# Using brand.yml with Quarto

A practical guide for integrating `_brand.yml` into Quarto projects to achieve unified visual styling across multiple output formats.

## Overview

[[concepts/quarto]] automatically discovers and applies `_brand.yml` from the project root, requiring no explicit configuration in most cases. Supported formats include HTML documents, HTML dashboards, RevealJS presentations, Typst PDFs, and multi-page websites.

## Key Concepts

### Automatic Discovery
Placing `_brand.yml` at the project root is sufficient — Quarto detects and applies it to all supported output formats without additional configuration.

### Document vs. Project Level
- **Document-level**: Set `brand: _brand.yml` in the [[concepts/yaml-configuration]] frontmatter under the format key.
- **Project-level**: Set `brand: _brand.yml` under the `project:` key in `_quarto.yml`.
- **Custom paths**: Any path can be specified via the `brand:` field.

### Brand Integration
Quarto reads the [[concepts/brand-theming]] file's `color`, `typography`, `logo`, and `meta` sections and maps them to format-specific styling systems. The [[concepts/brand-color-system]] and [[concepts/brand-typography]] definitions drive visual consistency across all output formats.

### Theme Layering
The `brand` keyword can be inserted into the `theme:` list to control precedence:
- Lowest priority (default, omit `brand`): external themes override brand
- Highest priority: place `brand` last in the list
- Middle priority: place `brand` between other themes

### Light/Dark Mode
Any color or typography property in `_brand.yml` can have `light:` and `dark:` variants, enabling automatic adaptation for websites with theme toggles.

```yaml
color:
  foreground:
    light: "#333333"
    dark: "#e0e0e0"
```

## Format-Specific Features

### HTML / Websites
- Brand colors become Sass variables: `$brand-{color-name}`, as part of the [[concepts/brand-color-system]]
- Accessible in custom SCSS files
- Shortcodes (via Quarto extensions) expose brand values inline: `{< brand-color primary >}`

### Typst PDF
- Colors accessible as `brand-color.{name}` and `brand-background-color.{name}`
- Supported [[concepts/brand-typography]] elements: base, headings, title, monospace-inline, monospace-block, link
- Google Fonts auto-downloaded and cached at `.quarto/typst-font-cache/`
- Debug font issues with `#set text(fallback: false)`

### RevealJS
- Theme layering via `theme:` list with optional `brand` keyword
- Logo and brand colors applied to slides

## Brand Extensions

Reusable brand packages can be created as Quarto extensions:

```bash
quarto create extension brand
quarto add username/my-brand-extension
```

Extensions bundle `brand.yml`, logos, and fonts, declared via `contributes.brand` in `_extension.yml`. Requires a `_quarto.yml` project file.

## Sample Minimal `_brand.yml`

```yaml
color:
  palette:
    brand-blue: "#0066cc"
  primary: brand-blue
  background: "#ffffff"
typography:
  fonts:
    - family: Inter
      source: google
      weight: [400, 600]
  base:
    family: Inter
    size: 1rem
    line-height: 1.6
  headings:
    weight: 600
```

## Troubleshooting

| Problem | Fix |
|---|---|
| Brand not applying | Use underscore prefix `_brand.yml`; ensure `_quarto.yml` exists |
| Colors not matching | Quote hex values; check palette references |
| Fonts not loading | Verify Google Fonts spelling; check `source: google` |
| Typst font issues | Check `.quarto/typst-font-cache/`; use `fallback: false` |
| Extension not working | Verify `contributes.brand` in `_extension.yml` |

## Related Concepts
- [[concepts/documentation-as-code]]
- [[concepts/plain-text-documentation]]
- [[concepts/preview-deployments]]
- [[concepts/brand-theming]] — The brand definition file format and project-wide theming
- [[concepts/brand-color-system]] — Color palette and semantic color roles
- [[concepts/brand-typography]] — Typography settings across elements
- [[concepts/quarto]] — The Quarto publishing system
- [[concepts/yaml-configuration]] — YAML-based project and document configuration
- [[concepts/r-visualization-theming]] — Related theming in R-based outputs