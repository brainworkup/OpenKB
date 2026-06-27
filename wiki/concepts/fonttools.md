---
sources: [summaries/entry_points.md]
brief: fontTools is a Python library for reading, writing, and manipulating font files via CLI tools.
---

# fontTools: Python Font Manipulation Library

fontTools is an open-source Python library that provides tools for reading, writing, and transforming font files in various formats (TrueType, OpenType, etc.). It serves both as a Python API and as a suite of command-line utilities installed via [[concepts/cli-entry-points]].

## Overview

fontTools is widely used in font development pipelines, web font generation, and typography tooling. It underpins much of the modern font build infrastructure, including Google Fonts and other large-scale font projects.

## Command-Line Entry Points

When installed, fontTools registers four console scripts via [[concepts/python-entry-points]]:

| Command | Purpose |
|---|---|
| `fonttools` | Main CLI dispatcher (`fontTools.__main__:main`) |
| `pyftmerge` | Merges multiple font files into one |
| `pyftsubset` | Creates font subsets by removing unused glyphs |
| `ttx` | Converts fonts to/from XML (TTX format) |

See [[summaries/entry_points]] for the raw entry point declarations.

## Key Tools

### `ttx`
The `ttx` tool converts binary font files (`.ttf`, `.otf`) into a human-readable XML format (`.ttx`) and back. This enables inspection and manual editing of font internals, including glyph data, kerning, and OpenType feature tables. See [[concepts/fonttools-table-converters]] for details on how font tables are handled.

### `pyftsubset`
Subsetting removes glyphs and features that are not needed for a specific use case (e.g., a web page only using Latin characters). This dramatically reduces font file sizes for web delivery.

### `pyftmerge`
Merging combines multiple font files — for example, joining a base Latin font with a CJK supplement — into a single font binary.

### `fonttools` CLI
The main dispatcher provides access to a wide range of sub-commands for inspecting and transforming font data programmatically.

## Relationship to Other Concepts

- **[[concepts/cli-entry-points]]** — fontTools demonstrates the standard Python pattern of declaring `[console_scripts]` entry points in package metadata.
- **[[concepts/python-entry-points]]** — The package uses entry points to expose tools to the system PATH upon installation.
- **[[concepts/python-project-structure]]** — fontTools follows conventional Python package layout with module-level `main()` functions as entry targets.
- **[[concepts/open-source-licensing]]** — fontTools is released under an open-source license.
- **[[concepts/mypyc]]** — Parts of fontTools are compiled with mypyc for performance.

## Installation

```bash
pip install fonttools
```

After installation, `fonttools`, `pyftmerge`, `pyftsubset`, and `ttx` are available as shell commands.

## References
- [[summaries/entry_points]] — Entry point declarations for the fontTools package
