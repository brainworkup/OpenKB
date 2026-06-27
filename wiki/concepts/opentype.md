---
sources: [summaries/top_level.md]
brief: OpenType is a scalable font file format standard supporting advanced typography and variable fonts.
---

# OpenType Font Format

OpenType is a cross-platform scalable font file format developed by Microsoft and Adobe, now maintained as an open standard by the [OpenType specification](https://docs.microsoft.com/en-us/typography/opentype/spec/). It is the dominant font format used in modern operating systems, browsers, and design tools.

## Structure

An OpenType font file (`.otf`, `.ttf`, `.woff`, `.woff2`) is a collection of named binary tables, each storing a specific aspect of font data:

- **`glyf`** — TrueType glyph outlines (quadratic Bézier curves)
- **`CFF`/`CFF2`** — PostScript glyph outlines (cubic Bézier curves)
- **`cmap`** — Character-to-glyph mapping (Unicode support)
- **`GSUB`** — Glyph substitution (ligatures, alternates)
- **`GPOS`** — Glyph positioning (kerning, mark placement)
- **`GDEF`** — Glyph definition classes
- **`fvar`**, **`gvar`**, **`HVAR`** — Variable font axes and deltas
- **`head`**, **`hhea`**, **`OS/2`** — Font metadata and metrics

## Advanced Typography

OpenType supports sophisticated typographic features through its layout tables (`GSUB`, `GPOS`), enabling:
- Ligatures and contextual alternates
- Small caps and old-style figures
- Complex script shaping (Arabic, Devanagari, etc.)
- Mark-to-base and mark-to-mark attachment

These are exposed via OpenType feature tags (e.g., `liga`, `kern`, `smcp`).

## Variable Fonts

[[concepts/variable-fonts]] extend OpenType with continuous design axes (weight, width, optical size, etc.), allowing a single font file to represent an entire type family through interpolation.

## Tooling

[[concepts/fonttools]] (the `fontTools` Python library) is the primary open-source toolkit for reading, writing, and manipulating OpenType fonts. It provides access to all font tables and supports both TrueType and CFF outlines. See [[summaries/top_level]] for a summary of the fontTools library.

Font subsetting — reducing a font to only the glyphs needed for a document or web page — is a key use case enabled by [[concepts/font-subsetting]].

## File Formats

| Format | Description |
|--------|-------------|
| `.ttf` / `.otf` | Desktop OpenType fonts |
| `.woff` | Web Open Font Format (compressed) |
| `.woff2` | WOFF2 (Brotli-compressed, smaller) |
| `.ttx` | XML representation used by fontTools |

## Related Concepts

- [[concepts/variable-fonts]] — OpenType font variation technology
- [[concepts/font-subsetting]] — Reducing font file size for web delivery
- [[concepts/fonttools]] — Python library for OpenType manipulation
- [[concepts/fonttools-table-converters]] — Tools for converting between font table representations
- [[concepts/mypyc]] — Used to compile parts of the fontTools library for performance
