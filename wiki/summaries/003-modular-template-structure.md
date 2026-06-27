---
doc_type: short
full_text: sources/003-modular-template-structure.md
---

# ADR 003: Modular Template Structure for Report Sections

## Overview

This Architecture Decision Record (ADR) documents the decision to implement a **modular template system** for neuropsychological reports using Quarto's include mechanism. The system assembles reports from discrete, numbered QMD section files orchestrated by a main template.

## Status

**Accepted**

## Problem Statement

Neuropsychological reports must support:
- Consistent structure across report types (pediatric, adult, forensic)
- Reusable sections (tests, behavioral observations, cognitive domains)
- Flexible inclusion based on data availability
- Easy maintenance and multi-author collaboration

## Decision

Use a **modular template system** where:
- A main `template.qmd` acts as the orchestrator
- Individual section files use a **numbered prefix system** (e.g., `_01-00_nse.qmd`, `_02-05_memory.qmd`)
- Sections are included via [[concepts/quarto]]'s `{{< include >}}` mechanism
- Shared variables are stored in `_variables.yml`

## Numbered Prefix System

The two-part numbering scheme encodes section hierarchy:

| Prefix | Section |
|--------|---------|
| `00-XX` | Tests and assessment battery |
| `01-00` | Neuropsychological Status Exam (NSE) |
| `01-01` | Behavioral observations |
| `02-XX` | Cognitive domains (memory, executive, ADHD, emotion) |
| `03-00` | DSM-5/ICD-10 diagnoses & SIRF |
| `03-01` | Recommendations |
| `03-02` | Signature |
| `03-03` | Appendix and consent forms |

First two digits = major section; last two digits = subsection ordering. New sections can be inserted without full renumbering.

## Rationale

**For modular design:**
- **Reusability**: Sections shared across report variants
- **Maintainability**: Isolated changes per section
- **Flexibility**: Add/remove sections without editing main template
- **Collaboration**: Parallel development by multiple team members
- **Testability**: Independent section validation

**Alternatives rejected:**
- Monolithic template — hard to maintain and collaborate
- Conditional sections in one file — still large and unwieldy
- Separate projects per report type — code duplication

## Consequences

| Type | Detail |
|------|--------|
| ✅ Positive | Easy per-section maintenance and updates |
| ✅ Positive | Reusable across report types |
| ✅ Positive | Clear, navigable structure |
| ❌ Negative | More files to manage |
| ❌ Negative | Requires understanding of Quarto include mechanism |
| ❌ Negative | Consistent variable naming required across all sections |

## Key Implementation Details

- Main template path: `style/templates/typst-report/template.qmd`
- Dynamic domain inclusion: `{{< include _domains_to_include.qmd >}}`
- Shared config: `_variables.yml`

## Related Concepts
- [[concepts/neuropsychological-assessment-pipeline]]
- [[concepts/typst-typesetting]]
- [[concepts/documentation-as-code]]

- [[concepts/modular-report-architecture]] — cross-document synthesis of modular design patterns
- [[concepts/quarto]] — Quarto-specific templating mechanisms
- [[concepts/clinical-report-structure]] — standard sections and organization of neuropsychological reports
- [[concepts/yaml-configuration]] — shared variable configuration via `_variables.yml`
- [[concepts/neuropsychological-reporting]] — broader context of neuropsychological report authoring
- [[concepts/cognitive-domains]] — the `02-XX` subsections covering memory, executive function, ADHD, and emotion