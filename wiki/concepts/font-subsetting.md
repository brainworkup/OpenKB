---
sources: [summaries/top_level.md]
brief: Font subsetting reduces font file size by retaining only glyphs and features needed for a target use case.
---

# Font Subsetting

Font subsetting is the process of creating a reduced version of a font file that contains only the glyphs, tables, and OpenType features required for a specific use case. This dramatically reduces file size, making it essential for web font delivery and embedded font scenarios.

## How It Works

A full font file may contain thousands of glyphs covering many scripts and languages. Subsetting analyzes the target text or character set and strips out:

- Glyphs not needed for the target character set
- Unused OpenType layout tables (GSUB, GPOS rules)
- Unreferenced font tables or metadata
- Hinting data (optionally)

The result is a significantly smaller font file that renders identically for the target content.

## fontTools and Subsetting

The [[summaries/top_level]] document describes fontTools, the primary Python library for font manipulation. fontTools ships a dedicated subsetting engine via the `pyftsubset` command-line tool and the `fontTools.subset` module.

Key capabilities of `fontTools.subset` include:

- **Unicode-based subsetting**: specify a list of Unicode code points or ranges to retain
- **Text-based subsetting**: provide sample text; only needed glyphs are kept
- **Feature retention**: selectively keep or drop OpenType features
- **Layout closure**: automatically retains glyphs referenced by substitution rules (e.g., ligatures, alternates) that are reachable from the kept glyph set
- **WOFF/WOFF2 output**: subset and recompress in a single step

## Relationship to Variable Fonts

Subsetting intersects with [[concepts/variable-fonts]] because variable fonts contain additional tables (`fvar`, `gvar`, `HVAR`, etc.) that must be handled carefully. fontTools supports subsetting variable fonts while preserving the variation axes, or alternatively instancing (pinning) specific axis values.

## Use Cases

- **Web fonts**: Load only the glyphs visible on a page, reducing bandwidth
- **PDF embedding**: Embed only used characters in exported documents
- **App bundles**: Minimize font assets shipped with mobile or desktop apps
- **Localization**: Deliver script-specific subsets to appropriate locales

## Related Concepts

- [[concepts/fonttools]] — the library providing the subsetting engine
- [[concepts/opentype]] — the font format standard that subsetting operates on
- [[concepts/variable-fonts]] — variable fonts require special handling during subsetting
- [[concepts/font-subsetting]] — this concept page
- [[summaries/table_API_readme]] — fontTools table API used internally by the subsetter
