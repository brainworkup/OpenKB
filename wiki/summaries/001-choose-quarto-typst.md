---
doc_type: short
full_text: sources/001-choose-quarto-typst.md
---

# ADR 001: Choose Quarto + Typst for Report Generation

## Overview

This Architecture Decision Record (ADR) documents the selection of **Quarto** combined with **Typst** as the report generation stack for the Voice project, which produces professional neuropsychological evaluation reports.

## Problem Context

The Voice project needed a solution capable of:
- Handling complex document structure (sections, headers, tables)
- Producing professional typography and formatting
- Supporting [[concepts/reproducible-builds]] across multiple environments
- Generating multiple report types: pediatric, adult, and forensic
- Integrating R code execution for data visualization
- Supporting version control and team collaboration

## Decision

**Quarto** was chosen as the document generation framework, with **Typst** as the underlying typesetting engine.

### Quarto Rationale
- Built on Pandoc with enhanced features
- Native R code chunk support via `knitr`
- Supports multiple output formats: PDF, HTML, Word
- Deep integration with the R ecosystem (ggplot2, dplyr), supporting [[concepts/r-python-integration]]
- [[concepts/yaml-configuration]]-based setup enabling [[concepts/reproducible-builds]]
- Active community and long-term support commitment

### Typst Rationale
- Modern alternative to LaTeX with faster compilation
- More intuitive syntax and better error messages
- Native Unicode support
- Growing ecosystem — see [[concepts/typst-typesetting]]

## Alternatives Rejected

| Alternative | Reason Rejected |
|---|---|
| LaTeX | Steep learning curve, slow compilation, complex errors |
| Pandoc alone | Less structured, harder to maintain complex templates |
| Word templates | Not reproducible, no code execution, poor version control |

## Consequences

**Positive:**
- Professional, reproducible PDF output
- Easy maintenance and version control
- Strong community backing

**Negative:**
- Learning curve for team members new to [[concepts/quarto]] and [[concepts/typst-typesetting]]
- Typst ecosystem is younger than LaTeX (fewer available packages)

## Implementation Details

- Quarto extensions created at `style/_extensions/brainworkup/`
- Typst templates implemented for each report type (pediatric, adult, forensic), supporting [[concepts/neuropsychological-reporting]]
- Project configured via `style/templates/typst-report/_quarto.yml`
- Variable substitution system using `_variables.yml`

## References
- [Quarto documentation](https://quarto.org/)
- [Typst documentation](https://typst.app/docs/)

## Related Concepts
- [[concepts/plain-text-documentation]]
- [[concepts/documentation-as-code]]
- [[concepts/clinical-report-structure]]
