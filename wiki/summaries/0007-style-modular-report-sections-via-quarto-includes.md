---
doc_type: short
full_text: sources/0007-style-modular-report-sections-via-quarto-includes.md
---

# Summary: Modular Report Sections via Quarto Includes (ADR-0007)

**Date:** 2025-01-20
**Status:** Accepted

## Overview

This Architecture Decision Record (ADR) establishes the use of **Quarto `{{< include >}}` shortcodes** to compose neuropsychological reports from modular `.qmd` section files. The approach addresses the complexity of reports that contain many semi-independent sections varying by patient and evaluation type.

## Problem Context

Neuropsychological reports include numerous sections:
- Test scores
- Behavioral observations
- Memory findings
- Executive function
- ADHD
- Emotion
- Diagnoses
- Recommendations
- Signature
- Appendix

These sections vary per patient and evaluation type, requiring a flexible composition mechanism.

## Decision

Use [[concepts/quarto]]'s `{{< include >}}` shortcode system to assemble the final report from modular `.qmd` files stored under `style/templates/typst-report/`. Key design elements:

- **`template.qmd`**: Main orchestration file controlling inclusion order.
- **Section files**: Each is self-contained with its own R setup chunks and [[concepts/typst-typesetting]] directives.
- **`_domains_to_include.qmd`**: A dispatcher file that dynamically includes domain sections based on available data.
- **Naming convention**: Numeric prefixes (`_00-`, `_01-`, `_02-`, `_03-`) enforce file system ordering and clarify section hierarchy (e.g., `_02-09_adhd.qmd`).

## Consequences

### Benefits
- **Section reuse**: Sections can be included or excluded per report without modifying shared template logic.
- **Parallel editing**: Multiple contributors can work on different sections simultaneously, minimizing merge conflicts.
- **Dynamic inclusion**: Data-driven dispatcher enables conditional section assembly.
- **Clear ordering**: Numeric prefixes make section hierarchy explicit in the file system.

### Trade-offs
- Each section must be self-contained, potentially leading to duplicated R setup code across files.
- Dispatcher logic in `_domains_to_include.qmd` must be maintained as new sections are added.

## Related Concepts
- [[concepts/narrative-report-generation]]
- [[concepts/neuropsychological-report-variables]]

- [[concepts/architecture-decision-records]] — Formal decision record format used for this ADR
- [[concepts/quarto]] — Quarto as the document generation framework
- [[concepts/modular-report-architecture]] — Pattern of assembling documents from reusable parts
- [[concepts/clinical-report-structure]] — Domain structure of clinical reports
- [[concepts/typst-typesetting]] — Typst used for PDF rendering within Quarto
- [[concepts/typst-modules]] — Typst module system relevant to per-section formatting
- [[concepts/neuropsychological-reporting]] — Overall context for clinical report generation
- [[concepts/quarto-extensions]] — Quarto extension ecosystem relevant to custom format inclusion