---
sources: [summaries/Apply-to-Y-Combinator-JWT.md, summaries/copilot-instructions.md, summaries/CLAUDE.md, summaries/2026-04-26-cingulate-agent-team-design.md, summaries/agent-team.md, summaries/DEPENDENCIES.md, summaries/0007-voice-modular-report-sections-via-quarto-includes.md, summaries/report-rendering-pipeline.md, summaries/brand-yml-integration.md, summaries/style-extensions.md, summaries/report-template.md, summaries/0010-voice-quarto-typst-reporting.md, summaries/0007-style-modular-report-sections-via-quarto-includes.md, summaries/0006-brand-yml-for-cross-platform-theming.md, summaries/0005-style-quarto-custom-format-extensions-for-report-variants.md, summaries/0001-voice-record-architecture-decisions.md, summaries/README.md, summaries/RECOVERY_NOTES.md, summaries/neuropsych-narrative-writer.md, summaries/responses_to_claude.md, summaries/SKILL.md, summaries/issue_branding_typst.md, summaries/report-generation.md, summaries/template-system.md, summaries/quarto-extensions.md, summaries/overview.md, summaries/003-modular-template-structure.md, summaries/001-choose-quarto-typst.md]
brief: Typst is the fast PDF typesetting engine behind Luria’s clinical report stack.
---

# Typst Typesetting

Typst is a modern typesetting system for producing high-quality, professionally formatted documents. In this knowledge base, it functions as the primary PDF rendering engine for Luria’s report generation stack, where it supports rapid iteration, readable templates, and precise layout control for clinical documents. It is used both through [[concepts/quarto]]-based report pipelines and through standalone `.typ` artifacts for specialized workflows.

Typst is especially valuable in Luria because the product aims to automate sensitive, high-stakes neuropsychological reporting while preserving professional presentation quality. In the YC application summary ([[summaries/Apply-to-Y-Combinator-JWT]]), Luria is described as a local-first, agent-based system that can execute much of the neuropsychological evaluation workflow end to end. Within that broader system, Typst is the final typesetting layer that turns structured clinical content into polished, clinician-facing output.

## Core Characteristics

- **Fast compilation**: Significantly faster than LaTeX, enabling rapid iteration on document design
- **Intuitive syntax**: Cleaner, more readable markup compared to LaTeX's macro-heavy language
- **Better error messages**: Human-readable diagnostics that reduce debugging time
- **Native Unicode support**: First-class handling of international characters and scripts
- **Growing ecosystem**: An expanding library of packages and community templates

## Why Typst Over LaTeX?

LaTeX has been the gold standard for professional typesetting for decades, particularly in academic and scientific publishing. However, it carries significant drawbacks:

| Feature | Typst | LaTeX |
|---|---|---|
| Compilation speed | Fast (incremental) | Slow (multi-pass) |
| Syntax clarity | Intuitive | Complex, macro-heavy |
| Error messages | Clear, readable | Cryptic, verbose |
| Unicode support | Native | Requires packages |
| Package ecosystem | Growing | Mature but large |
| Font configuration | System fonts directly | More complex |

For teams prioritizing developer experience and iteration speed — especially when building automated document pipelines — Typst's trade-offs are often favorable. ADR-0010 (see [[summaries/0010-voice-quarto-typst-reporting]]) formally consolidated this rationale, noting that Typst's rule/function model is more approachable than LaTeX macro-heavy templating and that faster compilation meaningfully improves the edit-render feedback loop for clinicians.

This matters in Luria’s context because the founder describes a workflow that evolved from R, RMarkdown, LaTeX, Quarto, and eventually Typst in response to real clinical reporting pressure. Typst represents the current endpoint of that progression: a rendering layer optimized for speed, maintainability, and professional output in a domain where reports are the primary deliverable.

## Role in Neuropsychological Report Generation

In the Voice and Luria stack, Typst serves as the typesetting engine underneath [[concepts/quarto]], handling the final rendering of professional neuropsychological evaluation reports. The architectural decision to use multiple Quarto custom format extensions — rather than a single monolithic template or one extension with conditional logic — reflects that the report variants share much of their layout logic but differ meaningfully in fonts, margins, paper size, and heading styles.

Typst templates are implemented for three distinct report types via Quarto extensions, each with its own `typst-template.typ` (layout) and `typst-show.typ` (data binding):

| Extension | Audience | Body Font | Heading Font | Paper Size | Font Size |
|---|---|---|---|---|---|
| `neurotyp-pediatric` | Children (under 18) | Equity B | Equity B (bold) | US Letter | 11.5pt |
| `neurotyp-adult` | Adults (18+) | Libertinus Serif | Libertinus Serif (bold) | US Letter | 11pt |
| `neurotyp-forensic` | Forensic/Legal | Libertinus Serif | Libertinus Sans (semibold) | A4 | 11pt |

All three extensions use APA citation style and do not number sections. The forensic extension additionally features customized list spacing, link styling, and optimized line breaks. This separation allows each report type to have tailored layout, typography, and structural conventions while sharing a common Typst-based rendering pipeline.

The extensions live under `style/_extensions/brainworkup/` within the broader report system. See [[summaries/001-choose-quarto-typst]] for the full architectural rationale, and [[summaries/0005-style-quarto-custom-format-extensions-for-report-variants]] for the ADR documenting the three-extension decision.

Within the larger Luria vision, Typst is not merely a formatting choice; it is part of a pipeline that aims to automate a neuropsychological evaluation workflow from intake and data extraction through synthesis and final report production. That makes Typst a core component of [[concepts/neuropsychological-assessment-automation]], [[concepts/neuropsychological-assessment-workflow]], and [[concepts/narrative-report-generation]].

## Quarto Extension Architecture

Each report type is packaged as a [[concepts/quarto-extensions]] Quarto extension. This packaging model means the Typst templates are not raw files applied directly — they are registered format contributions that Quarto loads and applies during rendering.

Every extension directory contains three files:

```text
style/_extensions/brainworkup/<report-type>/
├── _extension.yml
├── typst-show.typ
└── typst-template.typ
```

| File | Purpose |
|---|---|
| `_extension.yml` | Declares the format name, author, version, and `template-partials` |
| `typst-template.typ` | Defines the `#let report(...)` function — page setup, header, fonts, heading rules, logo, title block |
| `typst-show.typ` | Glues Quarto YAML variables to the `report()` function via `$if$` conditionals |

This two-file convention for `typst-template.typ` and `typst-show.typ` cleanly separates structural layout from element-level rendering and data binding, making each concern independently maintainable.

The `_extension.yml` file declares:
- Extension metadata (title, author, semantic version)
- Minimum required Quarto version (`>=1.4.0`)
- Format contributions: which Typst partials to load

Formats are selected in the project's `_quarto.yml` by referencing the extension name and can be further overridden per-project with custom papersize, font, or fontsize values:

```yaml
format:
  neurotyp-pediatric-typst:   # or neurotyp-adult-typst / neurotyp-forensic-typst
    keep-typ: true
    keep-md: true
    fontsize: 11.5pt
```

Extensions follow semantic versioning (MAJOR.MINOR.PATCH). The current development version for pediatric is `0.1.9999`; adult is `0.1.3`; forensic is `0.1.0`.

## Common Typst Patterns Across Extensions

All three templates share these structural and visual conventions:

- **Confidential header**: Shown on pages >1 with patient name and date in smallcaps
- **Run-in subheadings**: Level 4+ headings render as italic inline labels followed by a colon
- **Logo block**: Top of page 1 renders `inst/resources/logo.png` (or `img/logo.png` for adult/forensic)
- **Centered title**: `NEUROCOGNITIVE EXAMINATION` in 1.75em bold
- **Table styling**: `inset: 6pt`, `stroke: none`
- **Link styling**: Dark fill `rgb(4, 1, 23)`, weight 450, underlined

These shared conventions ensure visual consistency across [[concepts/neuropsychological-reporting]] regardless of clinical context.

## Margin and Typography Defaults

| Extension | Margins | Body Font | Heading Font |
|---|---|---|---|
| neurotyp-pediatric | top 30mm, right 25mm, bottom 30.25mm, left 25mm | Equity B | Equity B (bold) |
| neurotyp-adult | 1.25in × 1.25in | Libertinus Serif | Libertinus Serif (bold) |
| neurotyp-forensic | 25mm × 30mm | Libertinus Serif | Libertinus Sans (semibold) |

## Architecture Decision History

The choice of Typst as the PDF rendering engine was reached through an iterative ADR process. Two earlier ADRs recorded overlapping versions of the same decision — one broader, one focused on Typst-vs-LaTeX details. ADR-0010 ([[summaries/0010-voice-quarto-typst-reporting]]) consolidates the canonical rationale in a single place, confirming:

- [[concepts/quarto]] as the orchestration layer for authoring and data integration
- Typst as the primary PDF rendering engine
- Custom styling implemented via Typst templates and Quarto format extensions under `style/_extensions/brainworkup/`
- Quarto integration via `template-partials` (`typst-template.typ`, `typst-show.typ`) as used by the neurotyp extensions

The build process requires both Quarto and Typst to be installed on the clinician's machine. This is a known deployment dependency, balanced against the benefits of fast compilation and readable templates.

Historically, this choice also reflects the founder’s toolchain evolution from R and RMarkdown to LaTeX, then Quarto, then Typst, as the reporting workflow became more automated and more production-oriented. In that sense, Typst is part of a broader migration toward maintainable, programmable clinical documentation aligned with [[concepts/documentation-as-code]] and [[concepts/modular-report-architecture]].

## Variant Isolation vs. Shared Maintenance

A key consequence of the three-extension architecture is **variant isolation**: changes to one report type cannot accidentally break another. This is especially important in clinical contexts where layout regressions could affect document integrity.

The trade-off is a higher **maintenance surface**: three extensions mean three places to update for shared changes such as logo path or header format. Common patterns — including header rendering, run-in subheadings, and the confidential watermark — are currently duplicated across templates. A future refactor could extract these into shared [[concepts/typst-modules]] to reduce duplication while preserving isolation.

Users select the variant in `_quarto.yml` via `format: neurotyp-pediatric-typst` (or `-adult`, `-forensic`).

## Forensic Report Template (`forensic_report.typ`)

Beyond the Quarto extension system, a standalone Typst template — `forensic_report.typ` — is used to produce print-ready [[concepts/forensic-neuropsychological-evaluation]] reports independently of Quarto. This template is managed by the `typst-report-formatter` skill and is invoked as a parallel artifact alongside Google Docs report generation.

The template defines a `#let report(...)` function with the following parameters:

| Parameter | Description |
|---|---|
| `title` | Report title (default: "FORENSIC NEUROPSYCHOLOGICAL EVALUATION") |
| `patient` | Anonymized patient identifier |
| `case_number` | Case reference number |
| `date` | Date of examination |
| `body` | Main report content |

### Page and Typography Settings

- **Paper:** US Letter, 1.25in margins
- **Header:** Patient ID, case number, page number (right-aligned, 9pt gray)
- **Footer:** "Confidential Medicolegal Work Product" (centered, 8pt italic)
- **Body font:** Equity B, 11.5pt, justified paragraphs
- **Heading font:** Inter (bold 14pt for H1, semibold 12pt for H2)
- **H1 style:** Uppercase text with a full-width gray rule below

### Required Report Sections

All generated `.typ` files for forensic evaluations must include these top-level sections:

1. **Introduction and Purpose** — Referral reason and clinical questions
2. **Records Reviewed** — Bulleted list of reviewed records
3. **Background Information** — Demographics, history, presenting concerns
4. **Tests Administered** — Bulleted list of administered tests
5. **Neuropsychological Findings** — Subsections for General Cognitive Ability, Memory and Learning, Attention and Processing Speed, Executive Functioning, Language, and Visuospatial Ability
6. **Cognitive Profile Summary** — Integrated narrative
7. **Clinical Impressions and Diagnostic Formulation** — DSM-5/ICD-11 aligned impressions
8. **Recommendations** — Numbered list
9. **Limitations and Caveats** — Validity, effort, cultural factors
10. **Forensic Opinion** — Displayed in a shaded `#rect` box

### Score Tables

Structured test scores within domain sections are presented using Typst's `#table` function with five columns: Test/Subtest, Score, Score Type, Percentile, and Classification. This supports the score reporting conventions described in [[concepts/neuropsychological-test-scores]].

### PHI and Anonymization

The forensic template enforces strict [[concepts/phi-data-handling]] rules:

- **Never** use real patient names — always substitute `[PATIENT_ID]`
- Use `[CLINICIAN]` for evaluator references outside the fixed `Provider:` field
- Use `[CASE_NUMBER]` when the real case number is unknown or must be redacted
- The `Provider:` field is fixed as "Joey W. Trampush, Ph.D."

These rules align with the broader [[concepts/pii-redaction-pipelines]] and [[concepts/redaction-tokens]] conventions used across the pipeline.

The YC application reinforces why this design matters: the founder explicitly frames local handling of sensitive clinical material as non-negotiable and positions generic cloud AI tools as poorly suited to the privacy constraints of neuropsychological evaluation. Typst therefore fits naturally inside a [[concepts/local-first-architecture]], [[concepts/privacy-first-software]], and [[concepts/clinical-data-privacy]] strategy.

### Output and Compilation

The `typst-report-formatter` skill produces a single contiguous `.typ` file (template preamble → `#show` call → section content) and saves it to `/tmp/reports/[doc_id]_report.typ`. Because server-side Typst compilation is not available, the user must compile the file locally using the Typst CLI (`typst compile [file].typ`) or the Typst web app (https://typst.app). This positions the `.typ` file as a parallel, print-ready artifact alongside the [[concepts/narrative-report-generation]] Google Docs workflow.

## Integration with Quarto

Typst does not operate in isolation within this stack. [[concepts/quarto]] acts as the document authoring and orchestration layer, managing:

- R code chunk execution via `knitr`
- YAML-based configuration and variable substitution
- Multi-format output targeting (PDF primary; HTML and Word also supported)

Typst receives the processed document from Quarto and performs the final layout and PDF rendering. This clean separation of concerns — authoring vs. typesetting — is a key architectural strength of the chosen stack. The full data flow is:

```text
Raw PDF → MCP LLM Extraction → Structured Data (JSON/CSV)
  → Quarto Template → R/knitr Execution → Typst Rendering → Final PDF
```

For the forensic standalone path:

```text
Narrative Report Generator → typst-report-formatter skill → .typ file → User compiles locally → PDF
```

In the broader Luria workflow, this output stage sits downstream of [[concepts/pdf-data-extraction]], [[concepts/report-ingestion-pipeline]], [[concepts/neuropsychological-synthesis]], and [[concepts/clinical-narrative-generation]]. Typst itself does not perform reasoning; it renders the final artifact once upstream systems have assembled the content.

## Clinical Use Cases by Report Type

The three Quarto extensions encode distinct clinical conventions appropriate to their audiences:

- **Pediatric**: Developmental considerations, age-appropriate norms, educational implications, family-focused recommendations.
- **Adult**: Work-related assessments, disability evaluations, cognitive aging assessments, neurological condition evaluations.
- **Forensic**: Legal standards adherence, detailed methodology documentation, comprehensive disclaimer sections, expert witness testimony preparation.

The standalone `forensic_report.typ` template is an additional artifact targeting the forensic/legal audience, producing output consistent with [[concepts/forensic-neuropsychological-evaluation]] standards.

More generally, the founder’s YC application frames the underlying workflow as one where report writing is both the most labor-intensive step and the primary product delivered to patients, families, attorneys, and referring providers. That makes document quality central, not peripheral, to the system’s value proposition. Typst therefore supports not just formatting, but the operationalization of high-quality [[concepts/clinical-communication-register]] and [[concepts/clinical-report-structure]] at scale.

## Creating New Extensions

To add a new report type:

1. Create a directory under `style/_extensions/brainworkup/new-report-type/`
2. Author `_extension.yml` with format definition and metadata
3. Author `typst-template.typ` with document structure and page layout
4. Author `typst-show.typ` with element-level styling rules
5. Register the format in the project's `_quarto.yml`
6. Test with sample report data

## Reproducible Builds

Because Typst compilation is deterministic and fast, it supports reproducible document generation well. Given the same inputs (template + content + variables), Typst will produce consistent output — a critical requirement for versioned clinical documentation where audit trails matter.

This reproducibility is especially valuable in Luria’s envisioned end-to-end workflow, where many upstream steps may be automated or agent-assisted. Even if extraction, synthesis, and drafting involve AI components, the final rendering layer benefits from deterministic behavior.

## Troubleshooting

Common failure modes when working with Typst extensions:

- **Extension not found**: Verify path in `_quarto.yml`, check `_extension.yml` syntax, confirm Quarto version meets the `>=1.4.0` requirement.
- **Typst compilation errors**: Check `typst-template.typ` syntax, verify that the specified fonts are installed and available, review Typst error output.
- **Format not applied**: Confirm the format name in `_quarto.yml` matches the extension name exactly, and that template partials are correctly declared.
- **Standalone `.typ` file errors**: Ensure the template preamble (`#let report(...)`) is included verbatim before the `#show` call; verify font availability in the local environment.

## Considerations and Limitations

- **Ecosystem maturity**: Typst's package library is smaller than LaTeX's decades-old ecosystem. Some highly specialized LaTeX packages have no Typst equivalent yet.
- **Adoption curve**: Team members familiar with LaTeX or Word must learn Typst's syntax and conventions.
- **Community size**: While growing rapidly, the Typst community is younger than LaTeX's established user base.
- **Font dependency**: Each extension specifies particular fonts (Equity B, Libertinus Serif, Libertinus Sans) that must be available in the rendering environment. Typst uses system fonts directly, which simplifies configuration but requires fonts to be pre-installed on the build machine.
- **No server-side compilation**: The standalone forensic template must be compiled locally; there is no server-side Typst rendering available in the current pipeline.
- **Shared maintenance cost**: With three separate extensions, any cross-cutting change (logo path, header format, watermark) must be applied in three places until a shared module refactor is completed.
- **Dependent on upstream content quality**: Typst ensures presentation quality, but it does not solve extraction errors, synthesis mistakes, or weak clinical reasoning upstream in the pipeline.

## Related Concepts

- [[concepts/quarto]] — The document authoring framework that wraps Typst in the report stack
- [[concepts/quarto-extensions]] — The extension packaging model that registers Typst templates as Quarto formats
- [[concepts/typst-modules]] — Potential future refactor target for shared Typst logic across the three extensions
- [[concepts/neuropsychological-reporting]] — The domain context driving the need for professional typesetting
- [[concepts/forensic-neuropsychological-evaluation]] — The forensic evaluation workflow that the standalone template serves
- [[concepts/yaml-configuration]] — Used alongside Typst templates for variable substitution and project configuration
- [[concepts/documentation-as-code]] — The broader philosophy of treating documents as version-controlled, programmatically generated artifacts
- [[concepts/modular-report-architecture]] — The section-based template structure that Typst renders
- [[concepts/clinical-report-structure]] — The clinical conventions that Typst templates encode
- [[concepts/phi-data-handling]] — Anonymization and redaction rules enforced in the forensic template
- [[concepts/pii-redaction-pipelines]] — Broader redaction conventions aligned with template PHI rules
- [[concepts/redaction-tokens]] — Placeholder tokens (`[PATIENT_ID]`, `[CASE_NUMBER]`) used in forensic outputs
- [[concepts/narrative-report-generation]] — The Google Docs pipeline that runs in parallel with Typst output
- [[concepts/neuropsychological-test-scores]] — Score reporting conventions realized in Typst score tables
- [[concepts/architecture-decision-records]] — The ADR process that governs extension design decisions
- [[concepts/local-first-architecture]] — Architectural fit for keeping sensitive report generation local
- [[concepts/privacy-first-software]] — Broader product principle reinforced by local compilation and PHI constraints
- [[concepts/clinical-data-privacy]] — Clinical privacy context shaping template and deployment decisions
- [[concepts/neuropsychological-assessment-automation]] — The larger automated workflow that Typst helps finalize
- [[concepts/neuropsychological-assessment-workflow]] — The end-to-end workflow in which rendering is the final presentation step
- [[concepts/clinical-narrative-generation]] — Upstream narrative drafting layer whose outputs Typst renders
- [[concepts/neuropsychological-synthesis]] — Integrated interpretation layer rendered into final reports

## References

- [Typst official documentation](https://typst.app/docs/)
- [[summaries/0010-voice-quarto-typst-reporting]] — Consolidated ADR confirming Typst as the PDF rendering engine
- [[summaries/001-choose-quarto-typst]] — ADR documenting the selection of Typst for the reporting stack
- [[summaries/0005-style-quarto-custom-format-extensions-for-report-variants]] — ADR documenting the three-extension architecture decision
- [[summaries/style-extensions]] — Summary of the three neurotyp extension variants and their shared patterns
- [[summaries/quarto-extensions]] — Summary of the Quarto extension definitions for all three report types
- [[summaries/quarto]] — Related summary covering the Quarto framework
- [[summaries/overview]] — Component overview of the style system
- [[summaries/003-modular-template-structure]] — Modular template structure detail
- [[summaries/SKILL]] — The `typst-report-formatter` skill that governs standalone forensic `.typ` file generation
- [[summaries/template-system]]
- [[summaries/report-generation]]
- [[summaries/issue_branding_typst]]
- [[summaries/responses_to_claude]]
- [[summaries/neuropsych-narrative-writer]]
- [[summaries/RECOVERY_NOTES]]
- [[summaries/README]]
- [[summaries/0001-voice-record-architecture-decisions]]
- [[summaries/0006-brand-yml-for-cross-platform-theming]]
- [[summaries/0007-style-modular-report-sections-via-quarto-includes]]
- [[summaries/report-template]]
- [[summaries/Apply-to-Y-Combinator-JWT]] — Founder framing of Luria as a local-first agent-based clinical workflow system

See also: [[summaries/brand-yml-integration]]

See also: [[summaries/report-rendering-pipeline]]

See also: [[summaries/0007-voice-modular-report-sections-via-quarto-includes]]

See also: [[summaries/DEPENDENCIES]]

See also: [[summaries/agent-team]]

See also: [[summaries/2026-04-26-cingulate-agent-team-design]]

See also: [[summaries/CLAUDE]]

See also: [[summaries/copilot-instructions]]