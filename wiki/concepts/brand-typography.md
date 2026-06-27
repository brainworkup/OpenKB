---
sources: [summaries/report-rendering-pipeline.md, summaries/brand-yml-integration.md, summaries/style-extensions.md, summaries/brand-and-skills.md, summaries/0006-brand-yml-for-cross-platform-theming.md, summaries/brainworkup-branding-concepts.md, summaries/brainworkup-brand-voice-guide.md, summaries/quarto.md, summaries/brand-yml-spec.md]
brief: Structured font and typographic role specifications governing text appearance across all brand touchpoints and formats.
---

# Brand Typography Systems

A brand typography system is a structured specification of fonts and typographic rules that governs how text appears across all brand touchpoints — websites, applications, documents, and presentations. Rather than ad-hoc font choices, a typography system names fonts as reusable resources, maps them to semantic text roles, and encodes style decisions (weight, size, line height, color) in a single authoritative source.

In the [[concepts/brand-theming]] ecosystem, typography is one of the core pillars alongside color. The [[summaries/brand-yml-spec]] defines how typography is expressed in `_brand.yml` files used by Shiny, R, and Quarto. The [[summaries/quarto]] guide shows how these typographic specifications are automatically applied across HTML, RevealJS, and Typst PDF outputs.

Typography choices also carry communicative intent beyond aesthetics. The brainworkup brand voice guide illustrates how typeface selection reinforces brand personality: Merriweather (serif) for headings signals scholarly authority and tradition, while Atkinson Hyperlegible Next (sans-serif) for body text signals that clarity and accessibility are core values. JetBrains Mono handles clinical data display, reinforcing technical precision without visual noise. This principle — that typography mirrors voice — applies broadly to any brand system where audience trust and communication register matter.

## Core Components

### Font Definitions

Fonts are declared once in a `fonts` list and then referenced by name. This separation of *definition* from *use* prevents duplication and makes font swaps straightforward.

**Font sources supported:**

| Source | Description |
|---|---|
| `file` | Local or remote file paths, one entry per weight/style |
| `google` | Google Fonts CDN, weight arrays or variable ranges |
| `bunny` | GDPR-compliant Google Fonts alternative, same syntax |
| `system` | Operating system fonts, no download needed |

File-source fonts should be stored adjacent to the brand configuration file and referenced via relative paths. This is the recommended approach for proprietary typefaces.

```yaml
fonts:
  - family: Open Sans
    source: google
    weight: [400, 600, 700]
    style: [normal, italic]
  - family: Fira Code
    source: bunny
    weight: [400, 500]
```

### Weight Specification

Weights can be expressed in three ways:

- **Numeric array**: `[400, 700]`
- **Variable font range**: `400..900`
- **Named weights**: `[thin, normal, bold]`

### Semantic Text Roles

A typography system assigns fonts to *roles* rather than specific elements. This keeps the specification stable even as the underlying framework renders elements differently.

| Role | Purpose |
|---|---|
| `base` | Body/paragraph text |
| `headings` | All heading levels (h1–h6) |
| `monospace` | All code text (shorthand) |
| `monospace-inline` | Inline code spans |
| `monospace-block` | Fenced code blocks |
| `link` | Hyperlinks |

### Per-Element Style Properties

Each role accepts:

- `family` — must match a defined font family name
- `weight` — numeric or named
- `style` — `normal` or `italic`
- `size` — CSS units (`16px`, `1rem`, `0.9em`)
- `line-height` — number or CSS unit
- `color` — hex value or reference to a color palette name
- `background-color` — hex value or color name (useful for code blocks)
- `decoration` — link underline behavior

## Simple vs Extended Format

Roles can be defined minimally as a string (font family name) or fully as an object:

```yaml
# Simple
typography:
  base: Open Sans
  monospace: Fira Code

# Extended
typography:
  base:
    family: Open Sans
    size: 16px
    line-height: 1.5
  monospace-inline:
    color: "#7d12ba"
    background-color: "#f8f9fa"
```

The principle of preferring simple syntax applies: use a string when only the font family needs to be specified.

## Typography as Brand Signal

Typeface selection communicates personality and audience relationship before a single word is read. A typography system should reflect the communicative register of the brand:

- **Serif headings** (e.g., Merriweather) evoke authority, tradition, and scholarly credibility — appropriate for brands that need to establish trust with expert audiences.
- **Humanist or legibility-optimized sans-serifs** (e.g., Atkinson Hyperlegible Next) signal accessibility, clarity, and user respect — appropriate for body text reaching diverse or non-specialist readers.
- **Monospace fonts** (e.g., JetBrains Mono) signal technical precision and are best reserved for data, code, or clinical measure display rather than general prose.

This alignment between typeface character and [[concepts/brand-voice-strategy]] means typography decisions should not be made in isolation from messaging intent. A brand serving both anxious parents and forensic attorneys — as in the brainworkup example — may deliberately choose a serif/sans-serif pairing that bridges warmth and authority simultaneously.

When a brand employs dual-context communication (e.g., pediatric warmth vs. forensic precision), the typography system may use color palette switching alongside consistent typefaces to signal context shift, rather than changing fonts per audience. See [[concepts/brand-color-system]] for how dual palette systems (warm vs. authority) work alongside typography.

## Integration with Color Systems

Typography and [[concepts/brand-color-system]] are tightly coupled. Text roles can reference color palette names directly:

```yaml
typography:
  link:
    color: primary # References color.primary
  monospace-block:
    color: foreground
    background-color: background
```

This means a single change to `color.primary` automatically updates link colors everywhere. Base text color inherits from `color.foreground` by default and should not be overridden unless intentionally diverging.

## Format-Specific Typography Behavior

Typography specifications are interpreted differently depending on the output format.

### HTML and Web Formats

In HTML-based outputs (documents, dashboards, websites, RevealJS), brand font families are loaded from their declared source and applied via CSS. Custom SCSS can access brand typography through Sass variables. Google Fonts are fetched at build time.

### Typst PDF

When generating PDFs via Typst, Quarto automatically downloads and caches Google Fonts locally at `.quarto/typst-font-cache/`. Not all typography properties apply equally; the supported matrix is:

| Element | family | weight | color | background-color | line-height |
|---|---|---|---|---|---|
| base | ✓ | ✓ | ✓ | — | ✓ |
| headings | ✓ | ✓ | ✓ | — | ✓ |
| title | ✓ | ✓ | ✓ | — | ✓ |
| monospace-inline | ✓ | ✓ | ✓ | ✓ | — |
| monospace-block | ✓ | ✓ | ✓ | ✓ | ✓ |
| link | — | ✓ | ✓ | ✓ | — |

Font fallback behavior can be disabled in Typst templates with `#set text(fallback: false)`, which is useful for debugging font loading issues.

### R Visualizations

When used in R environments, brand typography can flow into ggplot2 and other visualization tools. See [[concepts/r-visualization-theming]] for how font choices propagate into data graphics.

## Light/Dark Typography Variants

Any typography color property can carry light and dark variants, enabling adaptation for theme-toggled interfaces:

```yaml
typography:
  link:
    color:
      light: "#0066cc"
      dark: "#3399ff"
```

This mirrors the light/dark pattern used in [[concepts/brand-color-system]] and is particularly relevant for Quarto websites that support theme switching.

## Design Principles

1. **Define once, reference many** — declare fonts in `fonts`, use them by name in roles
2. **Semantic over presentational** — target `headings`, not `h1` through `h6`
3. **Minimal specification** — only include properties that deviate from sensible defaults
4. **GDPR awareness** — prefer `bunny` over `google` when EU compliance matters
5. **Proprietary fonts** — download and store locally; use `file` source with relative paths
6. **Format awareness** — verify typography renders correctly across HTML, PDF, and presentation outputs, as support varies by format
7. **Voice alignment** — choose typefaces whose character matches the brand's communicative register and audience relationship
8. **Accessibility first** — prioritize legibility, especially for content reaching non-specialist or anxious readers

## Related Pages

- [[summaries/brand-yml-spec]] — Full specification for `_brand.yml` typography fields
- [[summaries/brand-yml-in-r]] — Using brand typography in R workflows
- [[summaries/quarto]] — Applying brand typography in Quarto multi-format publishing
- [[summaries/brainworkup-brand-voice-guide]] — Example of typography choices aligned with brand personality and dual-audience communication
- [[concepts/brand-color-system]] — The complementary color specification system
- [[concepts/brand-theming]] — Overarching brand configuration ecosystem
- [[concepts/brand-voice-strategy]] — How voice and tone decisions interact with visual identity
- [[concepts/yaml-configuration]] — YAML as a configuration language
- [[concepts/r-visualization-theming]] — Font propagation into R data graphics
- [[concepts/clinical-communication-register]] — How communication register (parent-facing, forensic, academic) shapes content and design decisions

See also: [[summaries/brainworkup-branding-concepts]]

See also: [[summaries/0006-brand-yml-for-cross-platform-theming]]

See also: [[summaries/brand-and-skills]]

See also: [[summaries/style-extensions]]

See also: [[summaries/brand-yml-integration]]

See also: [[summaries/report-rendering-pipeline]]