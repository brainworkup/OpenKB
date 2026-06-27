---
doc_type: short
full_text: sources/brand-yml-in-r.md
---

# Using brand.yml in R

## Overview

The `brand.yml` R package enables consistent brand styling across all R outputs — plots, tables, R Markdown documents, Quarto documents, and standalone scripts. It provides programmatic access to brand data and a set of theming functions for popular R visualization and table packages.

## Installation

```r
install.packages("brand.yml")
```

## Programmatic Access

Use `read_brand_yml()` to load brand data and access its properties directly:

```r
brand <- read_brand_yml()          # auto-discovers _brand.yml at project root
brand <- read_brand_yml("path")    # explicit path

brand$color$primary                # resolved color value
brand$color$palette                # full color palette list
brand$typography$base$family       # base font family
brand$meta$name                    # brand/company name
```

This enables dynamic color selection, conditional branding, and custom visualizations driven by brand data. The underlying file format is defined by the brand.yml specification (see [[concepts/yaml-configuration]] for general YAML configuration patterns).

## R Markdown Integration

Add brand.yml support to R Markdown HTML documents via YAML front matter:

```yaml
output:
  html_document:
    theme:
      version: 5      # Bootstrap 5 required
      brand: true     # or path to brand.yml
```

Bootstrap 5 (`version: 5`) is required for full brand.yml support.

## Quarto Integration

```yaml
format:
  html:
    brand: _brand.yml
```

Quarto-specific brand integration follows the same auto-discovery pattern, placing `_brand.yml` at the project root for seamless pickup.

## Theming Functions

All theming functions accept a `brand` parameter (NULL for auto-detect, file path, brand object, or FALSE) and color overrides (`background`, `foreground`, `accent`). See [[concepts/brand-theming]] for broader discussion of brand theming patterns.

### ggplot2

```r
ggplot(mtcars, aes(mpg, hp)) +
  geom_point() +
  theme_brand_ggplot2()
```

Additional parameters: `base_size`, `title_color`, `line_color`, `rect_fill`, `panel_background_fill`, `panel_grid_major_color`.

### gt Tables

```r
mtcars |> head() |> gt() |> theme_brand_gt()
```

### flextable

```r
mtcars |> head() |> flextable() |> theme_brand_flextable()
```

### plotly

```r
plot_ly(mtcars, x = ~mpg, y = ~hp, type = "scatter", mode = "markers") |>
  theme_brand_plotly()
```

Includes `accent` parameter (defaults to `brand.color.primary`).

### Base R Graphics (thematic)

Two modes:
- `theme_brand_thematic()` — returns a theme object for scoped use with `thematic::thematic_with_theme()`
- `theme_brand_thematic_on()` — immediately activates brand theming globally; disable with `thematic::thematic_off()`

## Common Patterns

- **Branded multi-visualization reports**: Load brand once, pass to multiple `theme_brand_*()` calls
- **Dynamic color palettes**: Extract `brand$color$primary/secondary/success` for custom plots
- **Conditional branding**: Switch brand files based on environment variables or context
- **Saving branded plots**: Use `ggsave()` after applying `theme_brand_ggplot2()`

## Key Tips

- Place `_brand.yml` at the **project root** for automatic discovery
- Set `version: 5` in R Markdown YAML for Bootstrap 5
- Combine theming functions across plots and tables for cohesive branded reports
- Test each theming function individually before combining in documents

## Related Concepts

- [[concepts/brand-theming]] — brand theming patterns and approaches
- [[concepts/r-visualization-theming]] — theming approaches for R visualizations
- [[concepts/yaml-configuration]] — YAML configuration patterns
- [[concepts/r-python-integration]] — broader R ecosystem integration patterns