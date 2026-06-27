---
sources: [summaries/README.md, summaries/report-rendering-pipeline.md, summaries/brand-yml-integration.md, summaries/brand-and-skills.md, summaries/0006-brand-yml-for-cross-platform-theming.md, summaries/0001-voice-record-architecture-decisions.md, summaries/issue_branding_typst.md, summaries/brainworkup-branding-concepts.md, summaries/quarto.md, summaries/brand-yml-spec.md, summaries/brand-yml-in-r.md, summaries/SKILL.md]
brief: Centralizing brand identity in _brand.yml for consistent styling across Quarto, Shiny, and Typst targets.
---

# Brand Theming for Data Applications

Brand theming for data applications refers to the practice of defining a single, centralized brand configuration that applies consistent colors, typography, and logos across interactive applications, visualizations, and documents. Rather than hardcoding visual styles into each individual app or report, a shared configuration file drives all styling decisions from one authoritative source.

## Core Concept

The central artifact is a `_brand.yml` file — a [[concepts/yaml-configuration]] document that encodes brand guidelines in a machine-readable format. Tools like Shiny (R and Python), Quarto, R Markdown, ggplot2, gt, flextable, and plotly consume this file to generate consistent themes automatically, eliminating manual CSS work and color hardcoding across projects.

This approach was formally adopted as an architecture decision (see [[summaries/0006-brand-yml-for-cross-platform-theming]]) to serve as the single source of truth for brand identity across Quarto documents, Shiny apps, and Typst PDFs — superseding the alternatives of duplicating theme config per target or relying solely on CSS/SCSS variables.

See [[summaries/brand-yml-spec]] for the complete field-by-field specification. See [[summaries/brand-yml-in-r]] for R-specific programmatic usage and theming functions. See [[summaries/quarto]] for Quarto-specific integration details. See [[summaries/brand-and-skills]] for the skill infrastructure supporting brand.yml authoring. See [[summaries/brand-yml-integration]] for a detailed map of how `_brand.yml` connects to each rendering target.

## Why Centralize Branding?

- **Consistency**: A single source of truth prevents color and font drift across projects
- **Maintainability**: Updating one file propagates changes everywhere
- **Separation of concerns**: Brand decisions live separately from application logic
- **Collaboration**: Designers can own the brand file without touching application code
- **Cross-platform reach**: A single file covers HTML apps, interactive documents, presentations, and PDFs

## File Naming, Location, and Discovery

Naming the file `_brand.yml` (with leading underscore) enables **automatic discovery** by Shiny and Quarto — no explicit paths needed:

- **Quarto ≥ 1.4** auto-discovers `_brand.yml` at the project root
- **Shiny ≥ 1.9** auto-discovers `_brand.yml` at the app root

The file is typically placed at the **project root**, or in `_brand/` or `brand/` subdirectories (e.g., `brand/_brand.yml`, referenced in `_quarto.yml` via `brand: brand/_brand.yml`). Custom names (e.g., `my-brand.yml`) are supported but require explicit configuration in each tool.

All top-level sections are **optional** — a valid `_brand.yml` can contain only the fields relevant to the brand.

## Current Project State

In the voice project, the `brand/` directory exists but is currently **empty** — no `_brand.yml` has been created yet. The `_quarto.yml` file references `brand: brand/_brand.yml`, which will fail until the file is created. The specification prompt lives at `brand-yml.prompt` at the project root, and the full skill definition and decision tree are at `skills/brand-yml/SKILL.md`.

## What a Brand Configuration Defines

### Meta
Optional company identity: `name` (simple string or `short`/`full` variant) and `link` entries for home, docs, GitHub, social platforms, etc. All links must include `https://`. Custom fields are allowed.

### Colors
A two-level system:
1. **Palette** — named raw values (e.g., `brand-blue: "#447099"`)
2. **Semantic colors** — Bootstrap-standard roles (e.g., `primary: brand-blue`)

Semantic names like `primary`, `secondary`, `success`, `warning`, `danger`, `info`, `foreground`, and `background` map directly to UI component styling. Semantic colors can reference palette names directly. Including standard Bootstrap color aliases (`blue`, `red`, `green`, etc.) ensures maximum compatibility with downstream tools.

Best practices:
- Use hex format: `"#447099"`
- Use lowercase hyphenated names following Sass conventions
- For shade ranges, choose the midpoint as the primary color
- Palette colors become Sass variables (`$brand-{color_name}`) in the `defaults` section

Light/dark mode variants are supported in Quarto 1.6+ and across the brand.yml ecosystem using structured color objects:
```yaml
primary:
  light: "#0066cc"
  dark: "#3399ff"
```
This enables websites and presentations to adapt automatically to user theme preferences.

See [[concepts/brand-color-system]] for detailed color strategy.

### Typography
Fonts are declared under `typography.fonts` with a `source` type:

| Source | Description |
|---|---|
| `file` | Local or remote file paths; weight and style per file |
| `google` | Google Fonts; weight arrays, ranges, or named values |
| `bunny` | GDPR-compliant alternative; same syntax as Google |
| `system` | System fonts |

Weight values can be numbers (`400`), arrays (`[400, 700]`), ranges (`400..900`), or named (`thin`, `normal`, `bold`).

Semantic typography roles — `base`, `headings`, `monospace`, `monospace-inline`, `monospace-block`, `link` — are assigned font families and styling properties. Each element supports `family`, `weight`, `style`, `size`, `line-height`, `color`, and `background-color`. Base text color defaults to `color.foreground`.

See [[concepts/brand-typography]] for broader typography system details.

### Logos
Multiple sizes (`small`, `medium`, `large`) and light/dark variants can be declared. Named resources defined in `logo.images` can be referenced by name in size slots, enabling responsive and theme-aware logo display:
```yaml
logo:
  images:
    header: logos/header-logo.png
    header-white: logos/header-logo-white.png
    icon: logos/icon.png
  small: icon
  medium:
    light: header
    dark: header-white
```
Alt text is supported via an object with `path` and `alt` keys. Paths can be local (relative to `_brand.yml`) or remote URLs.

### Defaults
The `defaults` section provides framework-specific overrides for Bootstrap/bslib (SCSS functions, variable overrides, mixins, rules), Quarto (format options), and Shiny (theme defaults and rules). Use sparingly — only when standard sections cannot meet brand requirements.

## Integration Targets

### Shiny for R
Uses `bslib::bs_theme(brand = TRUE)` for automatic discovery of `_brand.yml` at the app root (Shiny ≥ 1.9). Explicit paths are also supported via `bs_theme(brand = "path/to/_brand.yml")`. Part of the broader [[concepts/r-python-integration]] ecosystem.

### Shiny for Python
Uses `ui.Theme.from_brand(__file__)`. Requires the `shiny[theme]` extra (`pip install "shiny[theme]"`) and `libsass`. Auto-discovers `_brand.yml` when it is placed at the app root alongside the Python file.

### Quarto
Auto-discovery when `_brand.yml` co-exists with `_quarto.yml` at project root (Quarto ≥ 1.4). Configured explicitly via:

```yaml
project:
  type: default
  brand: brand/_brand.yml
```

Supports HTML documents, HTML dashboards, RevealJS presentations, Typst PDFs, and multi-page websites. Relates to [[concepts/documentation-as-code]] workflows where documents are generated from structured data.

**Theme layering**: Quarto allows the `brand` keyword in `theme:` lists to control precedence relative to other themes and custom SCSS:
```yaml
format:
  revealjs:
    theme:
      - cosmo
      - brand        # brand overrides cosmo
      - custom.scss  # custom.scss overrides brand
```

**Accessing brand values in documents**:
- Shortcodes (via Quarto extensions): `{< brand-color primary >}`
- SCSS variables: `$brand-primary`, `$brand-background`, `$brand-secondary`

**Typst PDF support**: Brand colors and typography apply to Typst-rendered PDFs via indirect translation rather than native application:
- `color.primary` → Typst `fill` values for headings/links
- `typography.base` → Typst `set text(font: ...)`
- `typography.headings` → Typst `show heading: set text(font: ...)`

Colors are available as `brand-color.{name}` and `brand-background-color.{name}`. Google Fonts are automatically downloaded and cached at `.quarto/typst-font-cache/`. **Important caveat**: Typst templates (e.g., in `style/_extensions/brainworkup/`) define their own font/paper defaults that may override brand.yml values depending on precedence, and not all brand.yml features have Typst equivalents. See [[summaries/issue_branding_typst]] for known limitations.

**Brand extensions**: Reusable brand packages can be bundled as Quarto extensions (bundling `brand.yml`, logos, and fonts), declared via `contributes.brand` in `_extension.yml`, and installed with `quarto add`.

### R Markdown
Brand styling is applied via YAML front matter:
```yaml
output:
  html_document:
    theme:
      version: 5      # Bootstrap 5 required
      brand: true     # or explicit path
```
Bootstrap 5 (`version: 5`) is required for full brand.yml support.

### R Visualization and Table Packages
The `brand.yml` R package provides dedicated theming functions for popular packages:

- **`theme_brand_ggplot2()`** — applies brand colors and fonts to ggplot2 plots
- **`theme_brand_gt()`** — themes gt tables with brand colors
- **`theme_brand_flextable()`** — themes flextable tables
- **`theme_brand_plotly()`** — applies brand colors to plotly charts
- **`theme_brand_thematic()`** — returns a scoped theme for base R graphics
- **`theme_brand_thematic_on()`** — activates brand theming globally for base R graphics

All functions accept a `brand` parameter (NULL for auto-detect, file path, brand object, or FALSE) and color overrides. See [[concepts/r-visualization-theming]] for broader context.

### Programmatic Access
Brand data can be read and accessed directly in any R script or document:
```r
brand <- read_brand_yml()           # auto-discovers _brand.yml
brand <- read_brand_yml("path")     # explicit path

brand$color$primary                 # resolved color value
brand$color$palette                 # full palette list
brand$typography$base$family        # base font family
brand$meta$name                     # brand/company name
```

This enables dynamic color selection, conditional branding (switching brand files based on environment variables), and custom visualizations driven by brand data.

## Creating brand.yml — Recommended Workflow

1. Gather brand info: colors, fonts, logos, company info
2. Read `references/brand-yml-spec.md` for the full specification
3. Build incrementally: colors → typography → logos → meta
4. Validate: YAML syntax, color references, font definitions, file paths
5. Test across all targets: Quarto HTML, Typst PDF, Shiny

## Skill-Driven Workflow

The `skills/brand-yml/` directory (part of the broader skills infrastructure) provides a decision tree and reference documentation for creating, modifying, and troubleshooting `_brand.yml` files. The decision tree covers the full lifecycle: creating from scratch → Shiny R integration → Shiny Python integration → Quarto integration → modifying existing files → troubleshooting.

This skill-driven approach — where reusable knowledge modules guide AI-assisted development — complements the [[concepts/skills-modules]] pattern and aligns with the [[concepts/architecture-decision-records]] philosophy of documenting both decision rationale and operational guidance.

The brand specification prompt lives at `brand-yml.prompt`, and the full skill definition is at `skills/brand-yml/SKILL.md`.

## Best Practices

| Practice | Rationale |
|---|---|
| Quote hex colors (`"#447099"`) | Prevents YAML parsing errors |
| Define palette before semantic colors | Enables references without forward declarations |
| Use lowercase hyphenated names | Sass naming conventions and YAML compatibility |
| Include Bootstrap color aliases | Picked up automatically by compatible tools |
| Prefer simple string syntax | Use strings over objects when possible |
| Start minimal, add incrementally | Easier to validate and debug |
| Version-control `_brand.yml` | Tracks brand changes alongside code |
| Place `_brand.yml` at project root | Enables automatic discovery by Quarto ≥1.4 and Shiny ≥1.9 |
| Set `version: 5` in R Markdown | Required for Bootstrap 5 and full brand support |
| Include `https://` in all URLs | Required for meta link fields |
| Test across output formats | Verify brand applies correctly to HTML, PDF, and presentations |
| Use light/dark variants for web | Enables automatic theme adaptation for interactive sites |
| Verify Typst output separately | Not all brand.yml features translate to Typst; template precedence may apply |

## Common Patterns

- **Color aliases**: Multiple palette names pointing to the same hex value, enabling both semantic and Bootstrap-standard access
- **Logo variants**: `images` map with named entries, then `light`/`dark` references per size slot
- **Multi-weight fonts**: Declared as arrays `[300, 400, 600, 700]` for design flexibility
- **Branded multi-visualization reports**: Load brand once with `read_brand_yml()`, pass to multiple `theme_brand_*()` calls for cohesive output
- **Conditional branding**: Switch brand files based on environment or deployment context
- **Dynamic color extraction**: Pull `brand$color$primary/secondary/success` for use in fully custom visualizations
- **Sass variable injection**: Reference `$brand-{color_name}` inside `defaults.bootstrap.rules` for low-level overrides
- **Quarto theme layering**: Insert `brand` keyword in the theme list to control precedence against other themes
- **Reusable brand extensions**: Package brand assets as a Quarto extension for multi-project consistency
- **Subdirectory placement**: Store as `brand/_brand.yml` and reference explicitly in `_quarto.yml` when project root placement is impractical

## Relationship to Adjacent Concepts

- [[concepts/yaml-configuration]]: `_brand.yml` is a domain-specific YAML schema
- [[concepts/documentation-as-code]]: Brand theming enables reproducible, consistently styled generated documents
- [[concepts/r-python-integration]]: The brand.yml ecosystem bridges R and Python toolchains with shared brand state
- [[concepts/plain-text-documentation]]: Like semantic linefeeds and plain-text docs, brand.yml keeps configuration human-readable and versionable
- [[concepts/r-visualization-theming]]: Theming functions for ggplot2, gt, flextable, plotly, and thematic all consume brand.yml data
- [[concepts/brand-color-system]]: Semantic color roles and palette design patterns
- [[concepts/brand-typography]]: Font sourcing, weight declarations, and typographic role assignment
- [[concepts/quarto]]: The Quarto publishing system that natively integrates brand.yml across output formats
- [[concepts/architecture-decision-records]]: Adoption of brand.yml was formalized as ADR 0006
- [[concepts/typst-typesetting]]: PDF rendering target with partial and indirect brand.yml support
- [[concepts/skills-modules]]: The brand-yml skill directory exemplifies reusable AI-assisted development knowledge modules

See also: [[summaries/brainworkup-branding-concepts]]

See also: [[summaries/issue_branding_typst]]

See also: [[summaries/0001-voice-record-architecture-decisions]]

See also: [[summaries/0006-brand-yml-for-cross-platform-theming]]

See also: [[summaries/brand-and-skills]]

See also: [[summaries/brand-yml-integration]]

See also: [[summaries/report-rendering-pipeline]]

See also: [[summaries/README]]