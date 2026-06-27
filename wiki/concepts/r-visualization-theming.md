---
sources: [summaries/report-rendering-pipeline.md, summaries/brand-yml-integration.md, summaries/brand-and-skills.md, summaries/0006-brand-yml-for-cross-platform-theming.md, summaries/quarto.md, summaries/brand-yml-spec.md, summaries/brand-yml-in-r.md]
brief: Applying consistent brand colors, fonts, and styles to R plots and tables across multiple packages.
---

# R Visualization Theming

R visualization theming refers to the practice of applying consistent visual styles — colors, fonts, backgrounds, and accents — across plots and tables produced by different R packages. Rather than manually setting colors in each chart or table, theming functions abstract brand configuration into reusable, composable wrappers.

## Why It Matters

R projects often produce outputs from multiple visualization libraries (ggplot2, plotly, base R graphics) and table packages (gt, flextable) within the same report or application. Without a shared theming system, visual inconsistency is common. A theming layer solves this by:

- Drawing colors and typography from a single source of truth (e.g., a `_brand.yml` file)
- Providing package-specific wrapper functions that apply those values correctly
- Allowing per-output overrides without abandoning the global brand

See [[summaries/brand-yml-in-r]] for a concrete implementation of this pattern using the `brand.yml` R package.

## Theming Function Pattern

Each theming function follows a consistent interface:

```r
theme_brand_<package>(
  brand = NULL,        # NULL = auto-detect, path, object, or FALSE
  background = ...,   # defaults to brand.color.background
  foreground = ...,   # defaults to brand.color.foreground
  accent = ...        # defaults to brand.color.primary (where applicable)
)
```

This pattern makes it easy to switch brands conditionally (e.g., internal vs. external), override individual colors, or pass a pre-loaded brand object.

## Package-Specific Implementations

### ggplot2 — `theme_brand_ggplot2()`
Extends the standard ggplot2 `theme()` system. Accepts fine-grained parameters including `base_size`, `title_color`, `line_color`, `rect_fill`, `panel_background_fill`, and `panel_grid_major_color`. Used as a drop-in addition to any `ggplot()` call.

### plotly — `theme_brand_plotly()`
Applied via the pipe operator to a `plot_ly()` object. Adds `accent` support for highlight/series colors in addition to background and foreground.

### gt — `theme_brand_gt()`
Themes gt table objects, controlling background and text color for table cells and headers.

### flextable — `theme_brand_flextable()`
Similar to `theme_brand_gt()` but for flextable output, which is commonly used in Word and PowerPoint-targeted reports.

### Base R / thematic — `theme_brand_thematic()` and `theme_brand_thematic_on()`
Two modes for theming base R graphics via the `thematic` package:
- **Scoped**: `theme_brand_thematic()` returns a theme object used within `thematic::thematic_with_theme()`
- **Global**: `theme_brand_thematic_on()` immediately activates theming for all subsequent plots; disabled with `thematic::thematic_off()`

## Integration with Brand Configuration

Theming functions are most powerful when backed by a structured brand configuration file. The `read_brand_yml()` function loads brand data and makes color palette values, typography settings, and metadata available programmatically:

```r
brand <- read_brand_yml()
my_colors <- c(brand$color$primary, brand$color$secondary, brand$color$success)
```

This connects visualization theming directly to [[concepts/brand-theming]] and [[concepts/yaml-configuration]] — the brand file acts as the single source of truth for all visual outputs.

## Scope of Application

Theming functions work uniformly across:
- **Standalone R scripts** (with `ggsave()` for export)
- **R Markdown documents** (HTML, with Bootstrap 5)
- **Quarto documents**
- **Shiny applications**

This cross-context consistency is a primary goal of the pattern. See [[concepts/r-python-integration]] for scenarios where themed R outputs are embedded in broader multi-language workflows.

## Common Patterns

### Multi-visualization branded report
```r
brand <- read_brand_yml()

plot1 <- ggplot(data1, aes(x, y)) + geom_point() + theme_brand_ggplot2()
plot2 <- ggplot(data2, aes(x, y)) + geom_line() + theme_brand_ggplot2()
table1 <- data3 |> gt() |> theme_brand_gt()
```

### Conditional brand switching
```r
brand <- if (Sys.getenv("BRAND_MODE") == "internal") {
  read_brand_yml("internal-brand.yml")
} else {
  read_brand_yml("external-brand.yml")
}
theme_brand_ggplot2(brand = brand)
```

## Key Principles

- **Single source of truth**: Brand colors and fonts live in one file, consumed by all theming functions
- **Auto-discovery**: Functions find `_brand.yml` at the project root automatically
- **Composability**: Multiple theming functions can be combined in a single document
- **Override-friendly**: Global defaults can be overridden per-output without losing brand coherence


See also: [[summaries/brand-yml-spec]]

See also: [[summaries/quarto]]

See also: [[summaries/0006-brand-yml-for-cross-platform-theming]]

See also: [[summaries/brand-and-skills]]

See also: [[summaries/brand-yml-integration]]

See also: [[summaries/report-rendering-pipeline]]