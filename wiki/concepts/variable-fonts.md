---
sources: [summaries/top_level.md]
brief: Variable fonts are OpenType fonts that encode multiple design variations in a single file using interpolation axes.
---

# Variable Fonts

Variable fonts are an OpenType specification that allows a single font file to contain an entire design space of variations — such as weight, width, slant, and optical size — rather than requiring separate files for each style. Introduced in OpenType 1.8, they enable continuous interpolation between defined extremes along one or more named axes.

## How They Work

Variable fonts encode variation data through a set of **axes** (e.g., `wght` for weight, `wdth` for width). Each axis has a minimum, default, and maximum value. Glyph outlines, metrics, and other font data are stored as **deltas** — differences from a default master — and interpolated at render time to produce any point along an axis.

### Key OpenType Tables
- `fvar` — Defines the variation axes and named instances
- `gvar` — Glyph variation data for TrueType outlines
- `CFF2` — Variation data for CFF (PostScript-style) outlines
- `HVAR` / `VVAR` — Horizontal and vertical metric variations
- `MVAR` — Miscellaneous metric variations
- `STAT` — Style attributes table, describing axis semantics

## Relationship to fontTools

The [[summaries/top_level]] document describes fontTools as a comprehensive Python library for font manipulation. fontTools provides extensive support for variable fonts through several modules:

- **`fontTools.varLib`** — Core tools for building variable fonts from multiple source masters, instancing (creating static snapshots), and merging variation data
- **`fontTools.designspaceLib`** — Reads and writes `.designspace` files that describe the design space, masters, and rules for variable font sources
- **`fontTools.ttLib`** — Low-level access to all variable font tables (`fvar`, `gvar`, `HVAR`, etc.)

## Design Space and Masters

Variable fonts are typically authored using a **designspace**, which defines:
- The axes and their ranges
- Source masters (fully designed extremes or intermediates)
- Rules for conditional substitution (e.g., swapping glyphs at extreme weights)

The [[concepts/opentype]] specification underpins the entire variable font system.

## Use Cases

- **Web typography** — A single variable font file replaces an entire font family, reducing HTTP requests and file size
- **Responsive design** — Typography adapts fluidly to screen size, resolution, or user preference
- **Animation** — Smooth transitions between weights or widths without layout reflow
- **Font production** — Type designers work with continuous design spaces rather than fixed masters

## Subsetting Variable Fonts

Variable fonts can be subsetted — reducing character coverage and axis ranges — using tools like `pyftsubset` from fontTools (see [[concepts/font-subsetting]]). Instancing produces a static font by fixing one or more axes at a specific value, which is useful for environments that do not support variable fonts natively.

## Related Concepts
- [[concepts/opentype]] — The specification that defines variable font tables
- [[concepts/font-subsetting]] — Reducing font file size, including axis range reduction
- [[concepts/fonttools]] — The Python library with primary tooling for variable font manipulation
- [[summaries/top_level]] — The fontTools top-level overview describing varLib and designspaceLib
