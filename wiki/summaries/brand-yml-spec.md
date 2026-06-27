---
doc_type: short
full_text: sources/brand-yml-spec.md
---

# brand.yml Specification

Complete reference for creating valid `_brand.yml` files, used by Shiny and Quarto for consistent brand configuration.

## File Naming & Location

- **Conventional name**: `_brand.yml` (auto-discovered by Shiny/Quarto)
- **Custom names**: Any `.yml` file with explicit paths
- **Location**: Project root, `_brand/`, or `brand/` subdirectories
- All top-level fields are **optional**

## Top-Level Sections

| Section | Purpose |
|---|---|
| `meta` | Company/project identity |
| `logo` | Logo files and variants |
| `color` | Color palette and semantic colors |
| `typography` | Fonts and text styling |
| `defaults` | Framework-specific customizations |

## Meta Section

Supports simple (`name: Acme`) or extended format with `short`/`full` name variants and multiple `link` entries (home, docs, github, bluesky, mastodon, linkedin, facebook, twitter). All links must use `https://`. Custom fields are allowed.

## Logo Section

- **Named resources**: `logo.images` maps names to file paths or URLs
- **Size variants**: `small`, `medium`, `large`
- **Light/Dark variants**: Each size accepts `light:` and `dark:` sub-keys
- **Alt text**: Supported via object format with `path` and `alt`
- Local files: relative paths from `_brand.yml`; remote: full URLs

## Color Section

See [[concepts/brand-color-system]] for detailed color strategy.

- **`color.palette`**: Named brand colors as flat hex map
- **Semantic colors**: `foreground`, `background`, `primary`, `secondary`, `tertiary`, `success`, `info`, `warning`, `danger`, `light`, `dark`
- Theme colors can reference palette names directly (e.g., `primary: brand-blue`)
- Use hex format: `"#447099"`
- Bootstrap color names preferred when applicable
- For shade ranges, choose midpoint as primary
- Palette colors become Sass variables: `$brand-{color_name}` in `defaults`

## Typography Section

See [[concepts/brand-typography]] for typography system details.

### Font Sources

| Source | Description |
|---|---|
| `file` | Local or remote file paths with weight/style per file |
| `google` | Google Fonts with weight arrays or ranges |
| `bunny` | GDPR-compliant alternative, same syntax as Google |
| `system` | System fonts |

### Typographic Elements

- `base` — body text
- `headings` — heading text
- `monospace` — all code text
- `monospace-inline` — inline code
- `monospace-block` — code blocks
- `link` — hyperlinks (supports `decoration` field)

Each element supports: `family`, `weight`, `style`, `size`, `line-height`, `color`, `background-color`.

Simple format accepts a string (font family name); extended format accepts an object. Base text color defaults to `color.foreground`.

Weight values: numbers (`400`), arrays (`[400, 700]`), ranges (`400..900`), or named (`thin`, `normal`, `bold`).

## Defaults Section

For framework-specific overrides only when standard sections are insufficient. See [[concepts/brand-theming]] for broader theming context.

- **`defaults.bootstrap`**: `functions`, `defaults` (variable overrides), `mixins`, `rules` (SCSS)
- **`defaults.quarto`**: Format-specific options
- **`defaults.shiny`**: Theme `defaults` and `rules`

## YAML Configuration

This file follows [[concepts/yaml-configuration]] conventions throughout. All fields are optional and the format favors simple string values over verbose object notation wherever possible.

## Validation Rules

1. All fields optional — include only what's needed
2. Prefer hex colors (`"#447099"`)
3. Prefer simple string syntax over objects
4. Sass naming: lowercase with hyphens
5. Always include `https://` in URLs
6. Define colors/fonts before referencing
7. Keep files concise

## Minimal Example

```yaml
color:
  palette:
    blue: "#0066cc"
    gray: "#666666"
  primary: blue
  foreground: gray
  background: "#ffffff"

typography:
  fonts:
    - family: Inter
      source: google
      weight: [400, 600]
  base: Inter
  headings:
    weight: 600
```

## Related Concepts
- [[concepts/r-visualization-theming]]
- [[concepts/r-python-integration]]
- [[concepts/documentation-as-code]]
- [[concepts/plain-text-documentation]]
