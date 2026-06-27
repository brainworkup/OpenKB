---
sources: [summaries/cerner-autotext.md, summaries/copilot-instructions.md, summaries/LLM_AGENT_MAP.md, summaries/README.md, summaries/CLAUDE.md, summaries/2026-04-26-cingulate-agent-team-design.md, summaries/0007-voice-modular-report-sections-via-quarto-includes.md, summaries/customization.md, summaries/style-training-to-report-drafting.md, summaries/report-rendering-pipeline.md, summaries/style-extensions.md, summaries/report-template.md, summaries/0010-voice-quarto-typst-reporting.md, summaries/0007-style-modular-report-sections-via-quarto-includes.md, summaries/0005-style-quarto-custom-format-extensions-for-report-variants.md, summaries/neuropsych-narrative-writer.md, summaries/SKILL.md, summaries/issue_branding_typst.md, summaries/report-generation.md, summaries/template-system.md, summaries/quarto-extensions.md, summaries/overview.md, summaries/003-modular-template-structure.md]
brief: Modular report architecture assembles reports from reusable section files.
---

# Modular Report Architecture

Modular report architecture is a design pattern that decomposes complex document templates into discrete, independently maintained section files assembled at render time. Rather than a single monolithic template, each section of a report lives in its own file and is composed together by an orchestrator. This pattern is foundational to the neuropsychological report generation pipeline described in [[summaries/report-generation]] and [[summaries/overview]]. It also generalizes to clinical documentation systems where structured templates are populated from reusable components or dynamic tokens, such as Cerner Autotext workflows for patient-specific note assembly.

## Core Principles

- **Separation of concerns**: Each file or reusable unit handles one report section only
- **Reusability**: Sections can be shared across multiple report variants
- **Composability**: An orchestrator assembles sections dynamically at render time
- **Isolated maintainability**: Changes to one section don't affect others
- **Independent testability**: Sections can be validated in isolation
- **Parallel editing**: Multiple contributors can work on different sections simultaneously with minimal merge conflicts
- **Structured variable injection**: Patient- or case-specific details are inserted through controlled variables, includes, or token systems rather than repeated manual editing

## Implementation in Neuropsychological Reporting

The canonical examples in this wiki are the [[summaries/003-modular-template-structure]] ADR, the [[summaries/0007-style-modular-report-sections-via-quarto-includes]] ADR, the [[summaries/0007-voice-modular-report-sections-via-quarto-includes]] ADR, the [[summaries/overview]] component overview, and the [[summaries/template-system]] reference implementation, all of which define a modular template system for neuropsychological reports using [[concepts/quarto]].

The decision to use Quarto's `{{< include >}}` shortcode mechanism reflects a deliberate architectural choice: the `{{< include >}}` shortcode composes the final report from modular `.qmd` section files, with a main `template.qmd` orchestrating inclusion order and a `_domains_to_include.qmd` dispatcher enabling data-driven conditional assembly. The full reference implementation is documented in [[summaries/report-template]].

The ADR-0007 decision was accepted on 2025-01-20 and applies to section files under `inst/quarto/templates/typst-report/`. Each section file is explicitly self-contained with its own R setup chunks and Typst formatting. This self-containment is the key property that enables both section reuse and parallel editing without merge conflicts.

This same architectural logic appears in EHR documentation contexts: a note can be treated as an assembled artifact built from reusable text scaffolds plus dynamic patient fields. In Cerner Autotext, bracketed tokens resolve at render time to patient names, pronouns, demographics, dates, provider identities, medication lists, allergies, and problem-list content. While not file-based like Quarto includes, these tokens serve a parallel role as modular placeholders within larger clinical templates.

### The `cingulate` Package Implementation

The `cingulate` R package (documented in [[summaries/CLAUDE]]) extends this pattern with a fully automated, R6-driven domain QMD generation pipeline. Where the base pattern relies on static include files, `cingulate` generates per-domain QMD includes dynamically from processed neuropsychological data, wiring the modular template architecture directly into the [[concepts/neuropsychological-assessment-pipeline]].

The `cingulate` pipeline is:
```
CSV (data-raw/csv/) → DuckDB/Parquet → R6 domain processors → per-domain QMD includes → Quarto/Typst → PDF
```

Domain QMD generation follows a two-step pattern:
1. **Text first**: `generate_domain_text_qmd()` writes `_02-XX_domain_text.qmd` (the narrative prose)
2. **QMD shell second**: the `_02-XX_domain.qmd` file `{{< include >}}`s the text file and adds tables/plots

This mirrors the broader architecture's separation between prose generation and data rendering, but automates it end-to-end via the `DomainProcessorR6` + `DomainProcessorFactoryR6` class pair. The factory dynamically detects which cognitive domains exist in the input data and instantiates only those processors, so the assembled report always reflects the actual assessment battery administered, never a fixed template.

**Stable domain numbering** is a hard constraint in `cingulate`: `01_iq`, `02_academics`, `03_verbal`, etc. Casual renumbering breaks the pipeline because both `_quarto.yml` and `template.qmd` depend on this ordering. This mirrors the two-part `XX-YY_sectionname.qmd` convention used throughout the broader modular architecture.

**Multi-rater domains** (ADHD `_02-09`, emotion/behavior `_02-10`) require special handling: `cingulate` produces separate per-rater files (self/parent/teacher). The `check_rater_data_exists()` / `check_domain_raters()` helpers must always be called before generating these files; child vs adult reports diverge in which raters are expected.

**Edit protection** is built in: `cingulate` checks whether a target QMD file has been hand-edited before regenerating it. This is the same [[concepts/edit-protection-pattern]] used by the narrative writer stage, preventing silent overwrite of manual clinical revisions.

### Structure

All templates reside under `style/templates/typst-report/` (or `inst/quarto/templates/typst-report/` depending on the package track). A main `template.qmd` acts as the orchestrator, pulling in numbered section files via Quarto's `{{< include >}}` mechanism:

```text
template.qmd                        ← orchestrator
_quarto.yml                         ← template-specific Quarto configuration
_variables.yml                      ← shared patient/case variables
config.yml                          ← global project configuration
_00-00_tests.qmd                    ← assessment battery and test list
_01-00_nse.qmd                      ← neurobehavioral status exam
_01-01_behav_obs.qmd                ← behavioral observations
_02-05_memory.qmd                   ← memory domain
_02-06_executive.qmd                ← executive function domain
_02-09_adhd.qmd                     ← ADHD assessment
_02-10_emotion.qmd                  ← emotion/psychopathology
_03-00_dsm5_icd10_dx.qmd            ← DSM-5/ICD-10 diagnoses
_03-00_sirf.qmd                     ← summary, impairments, recommendations, findings
_03-00_sirf_text.qmd                ← SIRF narrative text
_03-01_recs.qmd                     ← recommendations
_03-02_signature.qmd                ← report signature
_03-03_appendix.qmd                 ← appendix and consent forms
_03-03a_informed_consent.qmd        ← informed consent
_03-03b_examiner_qualifications.qmd ← examiner qualifications
_domains_to_include.qmd             ← dynamic domain inclusion controller
```

Each section file is **self-contained** with its own R setup chunks and Typst formatting. Each domain section file corresponds to a cognitive subdomain; the **narrative text** for those sections is produced separately by a dedicated prose-writing stage and injected as `_NN-XX_<domain>_text.qmd` includes. Score tables, plots, and other computed content are rendered by the R layer; the text files contain prose only.

A useful contrast is Cerner Autotext: instead of physically separated section files, documentation may use a single note template containing reusable placeholders such as patient name, pronouns, MRN, DOB, current date, provider fields, medication list, allergies, social history, and medical problems. In both architectures, the authoring surface is standardized while individualized data are injected at render time.

### Master Document Flow

`template.qmd` orchestrates assembly in a fixed order:

1. **YAML front matter** — patient name, exam date, report date
2. **R setup chunk** — loads the core internal package, configures knitr options, sets the LLM backend (Ollama), picks system fonts for SVG figures via `pick_font()`
3. **Typst header block** — case number, patient name, DOB/age, exam dates, report date
4. **Include chain**:
   - `_00-00_tests.qmd` → `_01-00_nse.qmd` → `_01-01_behav_obs.qmd`
   - Typst `= NEUROCOGNITIVE FINDINGS` heading
   - `_domains_to_include.qmd` (dynamic domain sections)
   - Page break
   - `_03-00_sirf.qmd` → `_03-00_sirf_text.qmd` → `_03-01_recs.qmd` → `_03-02_signature.qmd`
   - Page break
   - `_03-03_appendix.qmd`

### Section Organization Table

| Prefix | Section | Description |
|---|---|---|
| 00-00 | Tests | Assessment battery and test list |
| 01-00 | NSE | Neurobehavioral status exam |
| 01-01 | Behav Obs | Behavioral observations |
| 02-XX | Domains | Cognitive domain assessments |
| 03-00 | Diagnoses | DSM-5/ICD-10 diagnoses |
| 03-00 | SIRF | Summary, Impairments, Recommendations, Findings |
| 03-01 | Recs | Recommendations |
| 03-02 | Signature | Report signature |
| 03-03 | Appendix | Appendix and consent forms |

### Numbered Prefix Convention

Section files use a two-part numbering scheme `XX-YY_sectionname.qmd`:
- **First two digits (XX)**: Major section (00=header, 01=interview, 02=domains, 03=conclusions)
- **Last two digits (YY)**: Subsection ordering within the major section

This naming convention, enforced across the file system, serves two purposes: it makes the section hierarchy explicit and allows new sections to be inserted without renumbering the entire system. The prefix table is **stable**; `_quarto.yml` and `template.qmd` depend on the ordering and casual renumbering breaks the pipeline.

### Dynamic Inclusion

Sections can be conditionally included based on data availability via `_domains_to_include.qmd`, enabling the same template infrastructure to produce different report variants by simply changing which sections are assembled. The dispatcher includes domain sections based on available data, supporting the `--to neurotyp-pediatric-typst`, `--to neurotyp-adult-typst`, and `--to neurotyp-forensic-typst` render targets exposed at the command line.

A conceptually similar pattern appears in EHR note building, where dynamic tokens may be present or absent depending on what structured data exist in the chart. Cerner examples include pulling medication lists, allergies, social history, diagnosis/problem lists, and provider data. In that environment, conditional completeness depends on source chart data and token functionality rather than on file-inclusion logic.

## End-to-End Rendering Pipeline

The [[summaries/report-rendering-pipeline]] document describes the complete workflow from patient data to final PDF. The pipeline passes through two compilation stages:

1. **Quarto render** — reads `_quarto.yml` and `_variables.yml`, resolves the format extension, executes R chunks via knitr (data processing, figure generation), resolves `{{< include >}}` shortcodes to assemble the full document, and generates intermediate Typst source (`.typ`).
2. **Typst compile** — converts the `.typ` intermediate into a final PDF.

Key input files driving the pipeline:

- `_variables.yml` — patient-specific metadata injected via `{{< var key >}}` shortcodes
- `_quarto.yml` / `config.yml` — format selection and R/LLM configuration
- `brand/_brand.yml` — visual branding
- `_extensions/brainworkup/` — format-specific Typst templates (`typst-template.typ`, `typst-show.typ`)

The render command is:

```bash
quarto render style/templates/typst-report/template.qmd
```

Output artifacts:
- **PDF**: Final report in `style/templates/typst-report/` (or configured `output_dir`)
- **Typst source**: Retained if `keep-typ: true`
- **Markdown intermediate**: Retained if `keep-md: true`

In the `cingulate` package, the render is invoked via `quarto render` from the project root, driven by `_quarto.yml` and the `neurotyp-pediatric-typst` format from the project-local extension at `inst/quarto/_extensions/brainworkup/`. Adult and forensic variants live alongside under `inst/quarto/`.

In an EHR context, the analogous render step occurs when the note is generated and bracketed placeholders are resolved to current patient data. Cerner Autotext demonstrates this with name, pronoun, demographic, date, and care-team tokens, as well as structured content inserts like medications and allergies.

## Configuration Layer

Three configuration files drive the pipeline:

- **`_quarto.yml`**: Project config defining format targets with per-format font and paper settings
- **`_variables.yml`**: Patient demographics, clinician info, pronouns, diagnoses — injected via `{{< var key >}}` shortcodes throughout the document. See also [[concepts/neuropsychological-report-variables]]
- **`config.yml`**: Pipeline config for data I/O, processing flags, report format selection, and MCP/LLM endpoint settings

This three-file split separates rendering configuration ([[concepts/yaml-configuration]]) from patient-specific variable injection from runtime pipeline flags; each concern is isolated and independently editable.

The same general principle applies in EHR templating: template structure, available token vocabulary, and patient-specific values are distinct layers. Cerner Autotext effectively uses a controlled token namespace as a variable-injection system for note text.

### Variable Injection Details

| Source | Mechanism | Target |
| --- | --- | --- |
| `_variables.yml` | `{{< var key >}}` | QMD content, R chunks |
| `_variables.yml` | `{{< var key >}}` inside `` ```{=typst} `` | Typst header block |
| `_quarto.yml` format opts | Quarto extension system | `typst-show.typ` `$if$` conditionals |
| `config.yml` | R `yaml::read_yaml()` | R setup chunk (LLM backend, data paths) |

Comparable EHR token examples include patient full name, first/last name, salutation, pronoun set, MRN, DOB, current date, admission date, attending/referring physician, medication list, allergies, social history, and medical problems. One documented implementation detail from Cerner is that an age token may be non-functional, with DOB-based insertion preferred instead; this illustrates that modular placeholder systems need token-level validation just as section-based systems need include-level validation.

## R Runtime Dependencies

The setup chunk in `template.qmd` requires:

| Package | Role |
|---|---|
| Core internal package | Setup, data processing, LLM integration |
| `dplyr`, `readr`, `here`, `yaml` | Data wrangling |
| `ggplot2`, `svglite` | Figure rendering with document-matched fonts via `pick_font()` |

The `cingulate` package specifically depends on many heavy imports including DuckDB, Arrow, gt, and quarto. This has an important operational implication: `library(cingulate)` must **never** be auto-loaded at R startup (e.g., via `.Rprofile`) because the IDE R handshake will time out. Instead, use `devtools::load_all('.')` from the console after R has finished startup. Similarly, R6 generators register on package load, so `source()` of individual files will not work; `devtools::load_all('.')` is always required.

The `pick_font()` function detects available system fonts and configures `svglite` + `ggplot2` to match document fonts, ensuring SVG figures use the same typeface as the Typst body text. The LLM backend is configured for Ollama, reflecting the [[concepts/local-llm-inference]] approach used throughout the pipeline.

## Score-Type Handling

Footnotes and z-score conversions branch on whether a data row is `t_score`, `scaled_score`, or `standard_score`. In `cingulate`, `ScoreTypeCacheR6` caches this detection. When adding a new neuropsychological test to the system, its score type must be recognized by `R/score_type_utils.R` and `R/neuropsych_test_scoring.R`. This is relevant to the modular architecture because score-type detection drives both table footnote generation and the narrative quality ratings inserted into domain text files. See also [[concepts/neuropsychological-test-scores]].

## Report Type Variants

The system supports three primary report formats, each implemented as a custom Quarto extension under `style/_extensions/brainworkup/` (or `inst/quarto/_extensions/brainworkup/` in `cingulate`). Format selection is made in `_quarto.yml`:

```yaml
format:
  neurotyp-pediatric-typst:   # pediatric
  # neurotyp-adult-typst:     # adult
  # neurotyp-forensic-typst:  # forensic
```

| Extension | Audience | Font | Paper |
|---|---|---|---|
| `neurotyp-pediatric` | Children | Equity B | A4 |
| `neurotyp-adult` | Adults | Libertinus Serif/Sans | US Letter |
| `neurotyp-forensic` | Forensic | Libertinus Serif/Sans | US Letter |

Each extension provides a `typst-template.typ` and `typst-show.typ` for consistent [[concepts/typst-typesetting]] rendering. Adding a new report type means creating a new extension directory, defining `_extension.yml`, creating the Typst files, and registering the format in `_quarto.yml`. See also [[summaries/0005-style-quarto-custom-format-extensions-for-report-variants]].

## Prerequisites

- **Quarto** ≥1.4.0 (for Typst format support)
- **Typst** (bundled with Quarto or standalone)
- **R packages**: core internal package, `dplyr`, `readr`, `here`, `yaml`, `ggplot2`, `svglite`
- **System fonts**: Equity B (pediatric), Libertinus Serif/Sans (adult/forensic)
- **`cingulate` package**: loaded via `devtools::load_all('.')`, never `source()`

## Stage 3: Narrative Writer

The modular architecture formally separates prose generation from data rendering. Stage 3 of the Luria neuropsych pipeline — documented in [[summaries/neuropsych-narrative-writer]] — is dedicated entirely to writing the **per-domain narrative text** that populates `_NN-XX_<domain>_text.qmd` files. This is a strict division of responsibility:

- **Stage 3 (narrative writer)**: Produces hedged, APA-style clinical prose for each cognitive domain — no raw scores, no tables, no plots.
- **R layer / `cingulate`**: Renders score tables, figures and assembles the final Typst PDF.

In `cingulate`, narrative generation is handled by the `NeuropsychResultsR6` class and the LLM routing layer. Models are selected per task (`domain_summary`, `rating_scales`, `overall_summary`, `recommendations`, `differential_dx`, `quick_interpret`) and per performance mode (`development` / `balanced` / `production`) from `inst/config/ollama_models.yml`. Before running any LLM-dependent workflow, model availability must be verified with `check_available_models()`.

This separation between generated prose and injected structured data has a close analogue in clinical note templates. Cerner Autotext lets clinicians combine standard report language with dynamically inserted chart data, supporting efficient structured note writing while preserving individualized content.

### Narrative Include File Naming

For each domain, the narrative writer produces a file named `_NN-XX_<domain>_text.qmd` at the patient workspace path. The full domain prefix table is:

| Prefix | Domain |
| --- | --- |
| `_02-01_iq` | General Cognitive Ability / IQ |
| `_02-02_academics` | Academic / Achievement |
| `_02-03_verbal` | Verbal / Language |
| `_02-04_spatial` | Visuospatial / Visual-Construction |
| `_02-05_memory` | Memory & Learning |
| `_02-06_executive` | Executive Function |
| `_02-07_motor` | Sensorimotor |
| `_02-08_social` | Social Cognition |
| `_02-09_adhd` | ADHD (multi-rater: `_self`, `_parent`, `_teacher` variants) |
| `_02-10_emotion` | Emotion/Behavior (multi-rater: same per-rater pattern) |
| `_02-11_adaptive` | Adaptive Functioning |
| `_03-00_sirf` | Summary, Impressions, Recommendations & Formulation |
| `_03-01_recs` | Recommendations |

### Multi-Rater Domains

For ADHD (`_02-09_adhd`) and Emotion/Behavior (`_02-10_emotion`), both the narrative writer and `cingulate`'s `DomainProcessorR6` produce **one file per rater** present in the data — e.g., `_02-09_adhd_self_text.qmd`, `_02-09_adhd_parent_text.qmd`, `_02-09_adhd_teacher_text.qmd`. If a rater has no data, that file is skipped entirely; fabrication is prohibited. In `cingulate`, `check_rater_data_exists()` / `check_domain_raters()` must always be called before generating these files.

### Narrative Content Structure

For each subdomain, the narrative writer drafts 2–4 short paragraphs covering:
1. **Performance summary** — what was tested; qualitative range drawn verbatim from the `range` column of the extractor CSV (never invented)
2. **Pattern interpretation** — relative strengths/weaknesses, intra-test scatter, score-type discrepancies
3. **Functional implication** — one hedged sentence linking performance to everyday, academic, or occupational impact

Raw scores and percentiles are **never included in prose**; those are rendered by the R table layer.

### Edit-Protection Pattern

Before overwriting any existing `_text.qmd`, the writer reads the file first. If it contains content not derivable from the current data, the new draft is appended as a `<!-- DRAFT: ... -->` comment block rather than overwriting. This [[concepts/edit-protection-pattern]] ensures that manual clinical revisions are never silently lost. The same protection applies in `cingulate`'s QMD regeneration workflow.

An analogous issue exists in EHR token systems: a placeholder may be available but unreliable. The Cerner Autotext source notes that the age token is non-functional and DOB-based insertion should be preferred. In modular report architecture terms, this is a token-level quality constraint: placeholders should be validated, and known-bad dynamic fields should be replaced with more reliable alternatives.

### Voice and Style Rules

- Hedged, APA-style clinical prose: *"performance is consistent with…"*, *"results suggest…"*, *"indicates relative weakness in…"*
- Register adapts to `age_group`: pediatric, adult, or forensic
- Can mirror a style profile or exemplar snippets block if provided, but the extractor CSV is always the sole evidence base
- Markdown only (no raw HTML); Quarto-compatible
- No diagnoses, etiology, or prognosis unless explicitly present in the source extraction

## Typst Output as a Parallel Artifact

Beyond the Quarto-driven pipeline, the architecture also supports generating **standalone Typst (`.typ`) source files** as print-ready parallel artifacts — documented in [[summaries/SKILL]]. This is distinct from the Quarto-rendered pipeline: rather than being assembled by the Quarto orchestrator, a standalone `.typ` file is produced by a dedicated formatter skill invoked when the user explicitly requests a `.typ` file, Typst output, or a formatted forensic report.

The standalone Typst template (`forensic_report.typ`) defines a `#let report(...)` function encapsulating all page setup, typography, and header/footer logic. A fully-populated `.typ` file includes:

- **Page setup**: US Letter paper, 1.25in margins, right-aligned header (patient ID, case number, page number), centered confidential footer
- **Typography**: Equity B body font at 11.5pt, Inter for headings
- **Required sections**: Introduction and Purpose, Records Reviewed, Background Information, Tests Administered, domain-specific Neuropsychological Findings subsections, Cognitive Profile Summary, Clinical Impressions and Diagnostic Formulation, Recommendations, Limitations and Caveats, and a shaded Forensic Opinion block
- **Score tables**: Structured five-column tables using Typst's `#table` function

This standalone output is saved to `/tmp/reports/[doc_id]_report.typ` and requires local compilation via the Typst CLI (`typst compile [file].typ`) or the Typst web app.

### PHI/Anonymization in Typst Output

The standalone Typst skill enforces strict anonymization rules consistent with the broader [[concepts/phi-data-handling]] approach:
- Patient names are always replaced with `[PATIENT_ID]`
- The evaluator field is fixed as "Joey W. Trampush, Ph.D."
- Case numbers use `[CASE_NUMBER]` when unknown or redacted
- Any other clinician references use `[CLINICIAN]`

This aligns with the [[concepts/redaction-tokens]] and [[concepts/pii-redaction-pipelines]] patterns used throughout the pipeline.

## End-to-End Pipeline Integration

The modular template is the rendering stage within a broader multi-stage pipeline. The full pipeline, as described in [[summaries/report-generation]] and [[summaries/overview]], integrates data extraction, narrative generation, and final Typst PDF assembly. The [[concepts/architecture-decision-records]] formalizing this system include ADR-0003 (modular template structure), ADR-0005 (custom Quarto format extensions), and ADR-0007 (Quarto includes for section composition).

The Cerner Autotext example broadens this concept beyond Quarto and neuropsychology: modular report architecture can be understood more generally as the controlled assembly of documentation from reusable structural units plus dynamic patient variables. In one implementation, those units are section files and includes; in another, they are note scaffolds and render-time tokens. The common architectural goal is the same: standardized clinical writing with efficient personalization and fewer transcription errors.

See also: [[summaries/0005-style-quarto-custom-format-extensions-for-report-variants]], [[summaries/0007-style-modular-report-sections-via-quarto-includes]], [[summaries/0007-voice-modular-report-sections-via-quarto-includes]], [[concepts/quarto-extensions]], [[concepts/neuropsychological-reporting]], [[summaries/0010-voice-quarto-typst-reporting]], [[summaries/report-template]], [[summaries/report-rendering-pipeline]], [[summaries/style-extensions]]

See also: [[summaries/style-training-to-report-drafting]]

See also: [[summaries/customization]]

See also: [[summaries/2026-04-26-cingulate-agent-team-design]]

See also: [[summaries/CLAUDE]], [[concepts/domain-processor-pattern]], [[concepts/r6-class-architecture]]

See also: [[summaries/README]]

See also: [[summaries/LLM_AGENT_MAP]]

See also: [[summaries/copilot-instructions]]

## Related Documents
- [[summaries/cerner-autotext]]
