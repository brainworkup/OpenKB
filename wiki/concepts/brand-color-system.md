---
sources: [summaries/brand-yml-integration.md, summaries/brand-and-skills.md, summaries/0006-brand-yml-for-cross-platform-theming.md, summaries/brainworkup-branding-concepts.md, summaries/brainworkup-brand-voice-guide.md, summaries/quarto.md, summaries/brand-yml-spec.md]
brief: Structured approach to defining, naming, and applying brand colors consistently across digital products and contexts.
---

# Brand Color Systems

A brand color system is a structured approach to defining, naming, and applying colors consistently across digital products. Rather than scattering raw hex values throughout templates and stylesheets, a brand color system separates **palette definitions** from **semantic role assignments**, enabling both visual consistency and maintainable theming.

See [[summaries/brand-yml-spec]] for the canonical specification and [[concepts/brand-theming]] for how color systems fit into broader brand configuration. For a real-world clinical brand example, see [[summaries/brainworkup-brand-voice-guide]], which defines a dual-identity color system for pediatric and forensic neuropsychology contexts.

## Core Concepts

### 1. Color Palette

The palette is a flat, named map of all raw brand colors:

```yaml
color:
  palette:
    white: "#FFFFFF"
    black: "#151515"
    blue: "#447099"
    orange: "#EE6331"
    green: "#72994E"
    teal: "#419599"
    burgundy: "#9A4665"
```

- Each entry maps a **descriptive name** to a **hex color value**
- Names follow Sass conventions: lowercase, hyphen-separated (e.g., `brand-orange`, `success-green`)
- Bootstrap color names (`blue`, `red`, `green`, `teal`, `cyan`, etc.) are preferred when applicable
- For color ranges or shade families, choose the **midpoint** as the primary palette color

### 2. Semantic Color Roles

Semantic colors assign palette entries to functional UI roles, decoupling visual appearance from meaning:

| Role | Purpose |
|---|---|
| `foreground` | Main text color |
| `background` | Main background color |
| `primary` | Links, buttons, primary actions |
| `secondary` | Lighter text, disabled states |
| `tertiary` | Hover states, accents |
| `success` | Positive feedback |
| `info` | Neutral information |
| `warning` | Cautions |
| `danger` | Errors, destructive actions |
| `light` | High contrast on dark backgrounds |
| `dark` | High contrast on light backgrounds |

Semantic roles reference palette names rather than raw hex values:

```yaml
color:
  palette:
    brand-blue: "#447099"
    brand-orange: "#EE6331"
  primary: brand-blue
  warning: brand-orange
```

This indirection means a brand color change only requires updating the palette entry.

### 3. Color Aliases

Palette entries can alias other palette entries:

```yaml
color:
  palette:
    burgundy: "#9A4665"
    danger-color: burgundy  # alias
```

This is useful for maintaining both descriptive and semantic names within the palette.

### 4. Light/Dark Variants

Any color role can define separate values for light and dark modes, enabling automatic adaptation in environments with theme toggles (such as Quarto websites or dashboards):

```yaml
color:
  palette:
    blue: "#0066cc"
    navy: "#003366"
  foreground:
    light: navy
    dark: "#e0e0e0"
  background:
    light: "#ffffff"
    dark: "#1a1a1a"
  primary:
    light: "#0066cc"
    dark: "#3399ff"
```

Light/dark variants are especially important for multi-format publishing systems like [[concepts/quarto]], where the same brand file drives both web and PDF outputs.

## Dual-Identity Color Systems

Some brands require distinct color palettes for different audience contexts — not just light/dark adaptation, but fundamentally different emotional registers. The brainworkup brand (see [[summaries/brainworkup-brand-voice-guide]]) is a clear example:

**Warm palette (pediatric/family context):**
- Teal `#2A9D8F` — approachable, calming
- Coral `#E07A5F` — warm, encouraging
- Cream `#FAF3E0` — soft, inviting

**Authority palette (forensic/academic context):**
- Navy `#1B365D` — formal, trustworthy
- Gold `#C9A94E` — credentialed, serious
- Ivory `#FFFDF5` — clean, precise

**Shared neutrals (both contexts):**
- Charcoal `#2D2D2D`
- Slate `#4A6670`
- Ice `#F5F7FA`

This dual-identity approach mirrors the [[concepts/brand-voice-strategy]] principle that voice attributes stay constant while tone shifts by context. The color system encodes that same contextual flexibility: a single brand identity with palette variants that signal the appropriate register — warmth for families, authority for courts.

In a `_brand.yml` implementation, this could be expressed as context-specific palette groupings:

```yaml
color:
  palette:
    # Pediatric palette
    teal: "#2A9D8F"
    coral: "#E07A5F"
    cream: "#FAF3E0"
    # Forensic/academic palette
    navy: "#1B365D"
    gold: "#C9A94E"
    ivory: "#FFFDF5"
    # Shared neutrals
    charcoal: "#2D2D2D"
    slate: "#4A6670"
    ice: "#F5F7FA"
```

## Integration with Typography

Color systems interact directly with typography decisions. The brainworkup brand pairs its dual color palettes with typefaces that reinforce the same contextual split:
- Merriweather (serif headings) — scholarly authority, aligns with the forensic/academic palette
- Atkinson Hyperlegible Next (body) — accessibility priority, aligns with the pediatric/family palette

This alignment between color and type ensures visual consistency across channels. See [[concepts/brand-typography]] for how color roles are referenced in typography configuration.

## Integration with Frameworks

### Sass Variables

When using [[concepts/brand-theming]] with Bootstrap or Shiny, palette colors are automatically exposed as Sass variables using the pattern `$brand-{color_name}`. For example, a palette entry `orange: "#EE6331"` becomes `$brand-orange` in SCSS.

These variables can be used in the `defaults` section:

```yaml
defaults:
  bootstrap:
    defaults:
      navbar-bg: $brand-orange
```

In [[concepts/quarto]], brand colors are similarly available as Sass variables in custom SCSS files:

```scss
.branded-element {
  color: $brand-primary;
  background: $brand-background;
  border-color: $brand-secondary;
}
```

### Quarto Format Integration

Quarto maps the brand color system to multiple output formats from a single `_brand.yml` file:

- **HTML/RevealJS**: Colors become Sass/CSS variables applied via the theme pipeline
- **Typst PDFs**: Colors accessible as `brand-color.{name}` (palette colors) and `brand-background-color.{name}` (lighter background variants)
- **Dashboards**: Brand colors apply automatically to dashboard themes

Shortcodes (via Quarto extensions) can expose color values inline in documents:

```markdown
Our primary color is {< brand-color primary >}.
```

See [[summaries/quarto]] for a full breakdown of format-specific color integration.

### Typography Color References

Color palette names and semantic role names can be referenced directly in [[concepts/brand-typography]] settings:

```yaml
typography:
  monospace-block:
    color: foreground
    background-color: background
  link:
    color: primary
```

## Best Practices

1. **Always use hex format**: `"#447099"` — quoted to avoid YAML parsing issues
2. **Define palette first, assign roles second**: Establish all raw colors before semantic mapping
3. **Use Bootstrap-compatible names** where possible for better framework integration
4. **Keep the palette minimal**: Include only colors actually used in the brand
5. **Reference palette names in semantic roles** rather than repeating hex values
6. **Choose midpoints for shade ranges**: When a brand defines multiple shades of a hue, select the middle value as the canonical palette entry
7. **Add light/dark variants** for any color used in web contexts with theme toggles
8. **Align color palettes with tone registers**: In multi-context brands, design distinct palette groupings that match the emotional register of each audience context — just as voice tone shifts by channel, color can encode that same shift
9. **Pair color systems with typography choices** that reinforce the same contextual meaning

## Example: Complete Color Section

```yaml
color:
  palette:
    white: "#FFFFFF"
    black: "#151515"
    blue: "#447099"
    orange: "#EE6331"
    green: "#72994E"
    teal: "#419599"
    burgundy: "#9A4665"
  foreground: black
  background: white
  primary: blue
  success: green
  info: teal
  warning: orange
  danger: burgundy
```

With light/dark support:

```yaml
color:
  palette:
    blue: "#0066cc"
    navy: "#003366"
    gray: "#666666"
    light-gray: "#f5f5f5"
  primary: blue
  secondary: gray
  success: "#28a745"
  warning: "#ffc107"
  danger: "#dc3545"
  foreground:
    light: navy
    dark: "#e0e0e0"
  background:
    light: "#ffffff"
    dark: "#1a1a1a"
```

## Related Concepts

- [[concepts/brand-theming]] — How color systems integrate with full brand configuration
- [[concepts/brand-typography]] — Typography elements that reference color roles
- [[concepts/brand-voice-strategy]] — Voice and tone strategy that color systems can visually reinforce
- [[concepts/yaml-configuration]] — The YAML format underlying brand.yml files
- [[concepts/r-visualization-theming]] — Applying brand colors in R visualization contexts
- [[concepts/quarto]] — Multi-format publishing system consuming brand color definitions
- [[summaries/brainworkup-brand-voice-guide]] — Real-world dual-identity color system for clinical brand
- [[summaries/brand-yml-spec]] — Canonical specification for brand.yml color definitions


See also: [[summaries/brainworkup-branding-concepts]]

See also: [[summaries/0006-brand-yml-for-cross-platform-theming]]

See also: [[summaries/brand-and-skills]]

See also: [[summaries/brand-yml-integration]]