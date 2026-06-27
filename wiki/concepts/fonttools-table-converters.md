---
sources: [summaries/top_level.md, summaries/table_API_readme.md]
brief: Python classes in fontTools ttLib.tables that convert raw OpenType binary table data to Python objects and back.
---

# FontTools Table Converters

FontTools table converters are Python classes within the `ttLib.tables` subpackage of [[concepts/fonttools]] that translate raw TrueType/OpenType binary table data into Python objects and back. They are loaded automatically by `ttLib` when a font file is read or written, but can also be accessed directly via a documented API.

See [[summaries/table_API_readme]] for the original reference document.

## Context within fontTools

[[concepts/fonttools]] is a comprehensive Python library for manipulating font files (`.ttf`, `.otf`, `.woff`, `.woff2`). The table converter system is the low-level engine that makes it possible to read, modify, and write the dozens of named tables that constitute an [[concepts/opentype]] font. Higher-level modules such as `varLib` (for [[concepts/variable-fonts]]) and `feaLib` depend on table converters to access font internals.

## Tag-to-Identifier Encoding

OpenType/TrueType table tags are four-character, case-sensitive strings (e.g., `glyf`, `OS/2`, `cvt `). Because:
- File systems on Mac and Windows are case-insensitive, and
- Python identifiers only allow `[A-Za-z0-9_]` and cannot start with a digit,

…tags must be encoded before use as module or class names. This encoding scheme is described in [[concepts/python-identifier-encoding]].

### Encoding Rules
- Uppercase letters are separated by underscores: `FOO ` → `F_O_O_`
- Lowercase letters gain a leading underscore: `glyf` → `_g_l_y_f`
- Special characters like `/` become `f_`: `OS/2` → `O_S_2f_2`

### Conversion API
```python
from fontTools import ttLib
ttLib.tagToIdentifier("glyf")   # → '_g_l_y_f'
ttLib.identifierToTag("_g_l_y_f")  # → 'glyf'
```

## Module and Class Naming

Given a tag like `glyf`:
- **Module file:** `_g_l_y_f.py`
- **Class name:** `table__g_l_y_f` (prefix `table_` + expanded tag)

Convenience lookup functions:
```python
ttLib.getTableModule('glyf')   # returns the module
ttLib.getTableClass('glyf')    # returns the class
```

## Writing a Custom Converter

All converters must subclass `DefaultTable.DefaultTable`, which provides default behavior and an `__init__` constructor that does not need to be overridden.

### Required Methods

| Method | Purpose |
|---|---|
| `decompile(data, ttFont)` | Parse raw bytes into a Python data structure |
| `compile(ttFont)` | Serialize the Python data structure back to raw bytes |

### Optional TTX (XML) Methods

To support TTX import/export:

| Method | Purpose |
|---|---|
| `toXML(writer, ttFont)` | Serialize table data to XML |
| `fromXML((name, attrs, content), ttFont)` | Deserialize table data from XML |

### Memory Safety Warning

Do **not** store a reference to the `ttFont` argument as an instance attribute. `ttFont` already holds a reference to each table object, so storing back-references creates circular references and memory leaks. Pass `ttFont` as a method argument wherever needed.

## Relationship to Other Concepts

- [[concepts/fonttools]] — the parent library that provides the full font manipulation ecosystem
- [[concepts/opentype]] — the font format whose binary table structure these converters decode
- [[concepts/variable-fonts]] — OpenType font variations whose tables (e.g., `fvar`, `gvar`) are handled via this converter system
- [[concepts/python-identifier-encoding]] — the encoding scheme that maps table tags to valid Python identifiers and filenames
- [[concepts/python-project-structure]] — how subpackages like `ttLib.tables` are organized within a Python project
- [[concepts/documentation-as-code]] — the README-as-spec pattern used here to document converter conventions inline with the source
- [[concepts/font-subsetting]] — a key use case enabled by fontTools, relying on table converters to read and rewrite font tables

## Related Documents
- [[summaries/top_level]]
