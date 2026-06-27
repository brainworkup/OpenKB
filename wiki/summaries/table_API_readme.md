---
doc_type: short
full_text: sources/table_API_readme.md
---

# table_API_readme

## Overview
This document describes the conventions and API for writing custom TrueType/OpenType table converters within the `ttLib.tables` subpackage of the fontTools library. Converters translate raw binary table data to Python objects and back, and are loaded automatically by `ttLib` when needed.

## Tag-to-Identifier Mapping
TT/OT table tags are case-sensitive four-character strings (e.g., `glyf`, `OS/2`, `cvt `). Because file systems (Mac/Win) are case-insensitive and Python identifiers have restricted character sets (`[A-Za-z0-9_]`, no leading digit), tags must be encoded into a special expanded format. This encoding scheme is described in [[concepts/python-identifier-encoding]].

### Conversion Functions
- `ttLib.tagToIdentifier(tag)` — expands a tag to a safe identifier string
- `ttLib.identifierToTag(identifier)` — reverses the expansion

**Examples:**
```python
ttLib.tagToIdentifier("FOO ")  # → 'F_O_O_'
ttLib.tagToIdentifier("cvt ")  # → '_c_v_t'
ttLib.tagToIdentifier("OS/2")  # → 'O_S_2f_2'
ttLib.tagToIdentifier("glyf")  # → '_g_l_y_f'
```

The module file for the `glyf` table is therefore `_g_l_y_f.py`, and its class is `table__g_l_y_f`.

## Lookup API
Two convenience functions allow finding a table's module or class by tag directly:
```python
ttLib.getTableModule('glyf')   # → <module 'ttLib.tables._g_l_y_f'>
ttLib.getTableClass('glyf')    # → <class ttLib.tables._g_l_y_f.table__g_l_y_f>
```

## Writing a Custom Table Converter
All converters must subclass `DefaultTable.DefaultTable`, which provides default behavior and an `__init__` constructor. The naming conventions and plugin-style architecture relate to patterns described in [[concepts/fonttools-table-converters]].

### Required Methods
| Method | Purpose |
|---|---|
| `decompile(data, ttFont)` | Convert raw bytes into a Python data structure |
| `compile(ttFont)` | Convert the Python data structure back to raw bytes |

**Important:** Do not retain a reference to `ttFont` as an instance attribute — it creates a circular reference and causes memory leaks. Pass it as a method argument instead.

### Optional TTX Methods
To support XML import/export:
| Method | Purpose |
|---|---|
| `toXML(writer, ttFont)` | Serialize the table to TTX/XML |
| `fromXML((name, attrs, content), ttFont)` | Deserialize from TTX/XML |

## Key Concepts
- [[concepts/fonttools-table-converters]] — the broader naming and plugin conventions this README defines
- [[concepts/python-identifier-encoding]] — the tag-to-filename encoding scheme described here
- [[concepts/python-project-structure]] — relevant to how subpackages and modules are organized in Python libraries

## Related Concepts
- [[concepts/open-source-licensing]]
