---
doc_type: short
full_text: sources/0007-voice-modular-report-sections-via-quarto-includes.md
---

# Summary: Modular Report Sections via Quarto Includes (ADR-0007)

**Date:** 2025-01-20
**Status:** Accepted

## Overview

This Architecture Decision Record (ADR) establishes the use of **Quarto `{{< include >}}` shortcodes** to compose neuropsychological reports from modular `.qmd` section files. The decision addresses the complexity of reports that contain many semi-independent sections varying by patient and evaluation type.

## Problem Context

Neuropsychological reports consist of numerous sections:
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

These sections are not uniformly required across all patients or evaluation types, demanding a flexible composition mechanism.

## Decision

Adopt [[concepts/quarto]]'s `{{< include >}}` shortcode system to orchestrate report assembly:

- **Main template**: `template.qmd` controls the inclusion order of all section files.
- **Section files location**: `inst/quarto/templates/typst-report/`
- **Self-contained sections**: Each `.qmd` file includes its own R setup chunks and [[concepts/typst-typesetting]] directives.
- **Dispatcher file**: `_domains_to_include.qmd` dynamically includes domain sections based on available data.
- **Naming convention**: Numeric prefixes (`_00-`, `_01-`, `_02-`, `_03-`) enforce ordering and clarify hierarchy (e.g., `_02-09_adhd.qmd`).

## Key Consequences

### Benefits
- **Section reuse**: Sections can be included or excluded per report without touching shared template logic.
- **Parallel editing**: Contributors can work on different sections simultaneously, minimizing merge conflicts.
- **Dynamic inclusion**: The dispatcher pattern allows data-driven report composition.
- **Clear ordering**: Numeric file prefixes make section hierarchy visible in the file system.

### Considerations
- Each section must remain self-contained with its own setup, increasing some redundancy.
- The dispatcher file (`_domains_to_include.qmd`) becomes a critical coordination point.

## Related Concepts
- [[concepts/narrative-report-generation]]
- [[concepts/neuropsychological-report-variables]]

- [[concepts/architecture-decision-records]]
- [[concepts/modular-report-architecture]]
- [[concepts/neuropsychological-reporting]]
- [[concepts/clinical-report-structure]]
- [[concepts/quarto-extensions]]
- [[concepts/typst-modules]]