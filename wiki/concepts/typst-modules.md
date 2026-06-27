---
sources: [summaries/report-rendering-pipeline.md, summaries/report-template.md, summaries/0010-voice-quarto-typst-reporting.md, summaries/0005-style-quarto-custom-format-extensions-for-report-variants.md]
brief: Reusable Typst code modules that centralize shared layout logic across multiple document templates.
---

# Typst Modules and Shared Template Logic

In [[concepts/typst-typesetting]], a **module** is a reusable `.typ` file that encapsulates a discrete piece of layout or styling logic — fonts, heading styles, watermarks, header rendering — which can be imported by multiple document templates. This pattern reduces duplication and creates a single authoritative source for shared behavior.

## The Problem: Duplication Across Variants

When multiple document variants share the majority of their layout logic but differ in surface-level settings (fonts, margins, paper size), there is a structural choice to make:

1. **One template with conditional logic** — complex branching inside a single file
2. **Separate templates per variant** — clean isolation but duplicated shared code
3. **Separate templates + shared modules** — isolation with DRY (Don't Repeat Yourself) principles

The ADR documented in [[summaries/0005-style-quarto-custom-format-extensions-for-report-variants]] chose option 2 (separate extensions) as a pragmatic starting point, explicitly noting option 3 as a future refactor target.

## Shared Logic Candidates

In the neuropsychological reporting context, the following template elements are duplicated across the three report variants (`neurotyp-pediatric`, `neurotyp-adult`, `neurotyp-forensic`) and are natural candidates for extraction into shared Typst modules:

| Shared Element | Description |
|---|---|
| **Header rendering** | Clinic name, patient name, date block at top of each page |
| **Run-in subheadings** | Bold inline headings that flow with paragraph text |
| **Confidential watermark** | Background watermark applied to all pages |
| **Logo placement** | Consistent logo path and positioning logic |
| **Section dividers** | Horizontal rules or spacing between report sections |

## How Typst Modules Work

Typst supports modular composition via `#import`:

```typst
// In a shared module: shared/watermark.typ
#let confidential-watermark() = [
  // watermark implementation
]

// In a variant template: typst-template.typ
#import "../../shared/watermark.typ": confidential-watermark
#confidential-watermark()
```

This allows variant-specific files (`typst-template.typ`, `typst-show.typ`) to remain thin — handling only the variant-specific configuration — while delegating shared behavior to imported modules.

## Relationship to Quarto Extensions

In the [[concepts/quarto-extensions]] architecture, each Quarto custom format extension is a self-contained directory with its own Typst files. Shared Typst modules would live outside any individual extension directory (e.g., in a `style/_extensions/brainworkup/shared/` folder) and be referenced via relative import paths.

This creates a two-layer architecture:
- **Extension layer**: variant-specific configuration (`_extension.yml`, font choices, paper size)
- **Module layer**: shared layout functions imported by all extensions

## Trade-offs

### Benefits of Shared Modules
- **Single source of truth**: Fix a header rendering bug once, all variants benefit
- **Reduced maintenance surface**: Logo path changes in one place
- **Enforced consistency**: Shared visual elements remain identical across variants

### Risks
- **Coupling**: A breaking change to a shared module affects all variants simultaneously
- **Import path fragility**: Relative paths in Typst can break if directory structure changes
- **Premature abstraction**: If variants diverge significantly, forced sharing creates awkward conditionals inside modules

## Current State and Future Direction

As of the ADR decision date (2025-01-20), the three neuropsychological report extensions duplicate their common logic. The [[concepts/modular-report-architecture]] aspiration — extracting shared Typst modules — is identified as a future refactor. The immediate priority was variant isolation: ensuring that changes to the forensic report format cannot accidentally break the pediatric report.

This pattern mirrors broader software engineering principles applied to document engineering, where [[concepts/documentation-as-code]] practices encourage treating templates with the same modularity discipline as application code.

## Related Concepts
- [[concepts/typst-typesetting]] — The typesetting system in which modules are written
- [[concepts/quarto-extensions]] — The Quarto layer that wraps Typst templates
- [[concepts/modular-report-architecture]] — Broader principle of composable report components
- [[concepts/neuropsychological-reporting]] — Domain context for these templates
- [[concepts/documentation-as-code]] — Engineering discipline applied to document templates
- [[summaries/0005-style-quarto-custom-format-extensions-for-report-variants]] — Source ADR describing the current architecture and future refactor opportunity


See also: [[summaries/0010-voice-quarto-typst-reporting]]

See also: [[summaries/report-template]]

See also: [[summaries/report-rendering-pipeline]]