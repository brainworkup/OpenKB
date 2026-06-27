---
sources: [summaries/table_API_readme.md]
brief: A naming convention that encodes case-sensitive tags into valid, case-insensitive Python identifiers and filenames.
---

# Python Identifier Encoding for Case-Sensitive Tags

Python identifier encoding is a naming convention used in [[summaries/table_API_readme]] (the `ttLib.tables` subpackage of fontTools) to safely map case-sensitive four-character TrueType/OpenType table tags onto both case-insensitive file systems and valid Python identifiers.

## The Problem

OpenType/TrueType table tags are four-character, case-sensitive strings such as `glyf`, `OS/2`, `cvt `, and `FOO `. These tags must become:

1. **Filenames** on Mac and Windows, where `GLYF` and `glyf` refer to the same file.
2. **Python identifiers**, which may only contain `[A-Za-z0-9_]` and cannot begin with a digit.

A direct mapping is impossible — `OS/2` contains a slash, `cvt ` ends with a space, and simple lowercasing would lose case distinctions. A systematic encoding scheme is therefore required.

## The Encoding Rules

The encoding is implemented by `ttLib.tagToIdentifier()` and reversed by `ttLib.identifierToTag()`. The key transformations observed in the source are:

| Tag | Encoded Identifier |
|------|--------------------|
| `FOO ` | `F_O_O_` |
| `cvt ` | `_c_v_t` |
| `OS/2` | `O_S_2f_2` |
| `glyf` | `_g_l_y_f` |

The general pattern:
- Each character is separated by an underscore `_`.
- Lowercase letters are prefixed with an underscore `_` at the start (to disambiguate from uppercase).
- The `/` character is encoded as `f_` (representing the slash's ASCII relationship).
- Trailing spaces in the four-char tag become trailing underscores.

This ensures every unique tag produces a unique, reversible identifier regardless of case.

## Usage in fontTools

The encoded identifier is used as:
- **The filename** of the table converter module (e.g., `_g_l_y_f.py`)
- **Part of the class name**, prefixed with `table_` (e.g., `table__g_l_y_f`)

Developers rarely need to apply this manually. The `ttLib` API provides two convenience functions:

```python
from fontTools import ttLib

ttLib.getTableModule('glyf')   # finds _g_l_y_f.py automatically
ttLib.getTableClass('glyf')    # finds class table__g_l_y_f automatically
```

## Broader Applicability

This pattern is a specific instance of a general problem: encoding an arbitrary symbol space into a restricted identifier space in a **reversible, collision-free** way. Similar approaches appear in:

- URL percent-encoding (spaces and special chars → `%xx` sequences)
- C/Python name mangling for special characters
- Database column name sanitization

Any system that must bridge case-sensitive namespaces to case-insensitive storage, or arbitrary strings to restricted identifier syntax, benefits from a well-defined encoding and decoding pair.

## Related Pages

- [[summaries/table_API_readme]] — Source document describing this convention
- [[concepts/fonttools-table-converters]] — The broader table converter system this encoding supports
