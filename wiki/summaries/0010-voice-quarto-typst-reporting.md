---
doc_type: short
full_text: sources/0010-voice-quarto-typst-reporting.md
---

# Summary: Quarto and Typst for Neuropsychological Reporting (ADR-0010)

**Source:** `0010-voice-quarto-typst-reporting`
**Date:** 2026-04-22
**Status:** Accepted

## Overview

This Architecture Decision Record (ADR) consolidates the canonical choice of [[concepts/quarto]] as the authoring orchestration layer and [[concepts/typst-typesetting]] as the primary PDF rendering engine for neuropsychological evaluation reports. It supersedes two prior overlapping ADRs on the same topic. See also [[concepts/architecture-decision-records]] for the broader ADR practice in this project.

## Problem Context

[[concepts/neuropsychological-reporting]] requires:
- **Strong typographic control**: headers, page numbering, confidential markings, run-in subheadings
- **Modular authoring**: sections composed from separate source files, as described in [[concepts/modular-report-architecture]]
- **Multi-format output**: PDF (primary), HTML, and Word
- **AI/LLM-assisted drafting**: compatibility with [[concepts/narrative-report-generation]] workflows

The project prioritized fast iteration, template readability, and portable deployment on a clinician's machine.

## Decision

| Layer | Technology |
|---|---|
| Authoring & orchestration | **Quarto** (`.qmd` files) |
| PDF rendering | **Typst** (`.typ` templates) |
| Custom styling location | `style/_extensions/brainworkup/` |

## Rationale: Typst over LaTeX

| Factor | Typst | LaTeX |
|---|---|---|
| Compilation speed | ✅ Fast, single-pass | ❌ Slow, multi-pass |
| Template readability | ✅ Rule/function model | ❌ Macro-heavy |
| Font configuration | ✅ System fonts directly | More complex |
| Ecosystem maturity | ⚠️ Younger | ✅ Mature |

## Consequences

- Reports authored in `.qmd` (Quarto Markdown)
- Modular composition via Quarto include mechanisms and shared templates, as managed through [[concepts/quarto-extensions]]
- Typst templates define the visual layout system for PDF output, per [[concepts/typst-modules]]
- Build machine must have both **Quarto** and **Typst** installed
- Quarto integrates Typst via `template-partials` (`typst-template.typ`, `typst-show.typ`) used by the `neurotyp` extensions
- Fonts must be installed on the build machine (Typst uses system fonts directly)
- Missing Typst packages may require workarounds due to the younger ecosystem

## Related Concepts
- [[concepts/neuropsychological-assessment-pipeline]]

- [[concepts/quarto]] — Quarto as a multi-format scientific publishing system
- [[concepts/typst-typesetting]] — Typst as a modern programmable typesetting engine
- [[concepts/typst-modules]] — Modular Typst template components
- [[concepts/neuropsychological-reporting]] — Structure and requirements of neuropsychological evaluation reports
- [[concepts/modular-report-architecture]] — Composing reports from separate modular source files
- [[concepts/quarto-extensions]] — Custom Quarto format extensions for report variants
- [[concepts/clinical-report-structure]] — Clinical document structure and layout requirements
- [[concepts/narrative-report-generation]] — AI-assisted drafting of clinical narrative content