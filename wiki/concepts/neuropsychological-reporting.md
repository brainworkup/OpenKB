---
sources: [summaries/cerner-autotext.md, summaries/Luria_AI_Q4_Investor_Memo_2026.md, summaries/Apply-to-Y-Combinator-JWT.md, summaries/redesign_20260623110817.md, summaries/nt_interpretation.md, summaries/nse_narrative.md, summaries/neurobehav.prompt.md, summaries/LLM_AGENT_MAP.md, summaries/CLAUDE.md, summaries/agent-team.md, summaries/2026-04-26-cingulate-agent-team-design.md, summaries/0007-voice-modular-report-sections-via-quarto-includes.md, summaries/customization.md, summaries/style-training-to-report-drafting.md, summaries/report-rendering-pipeline.md, summaries/style-trainer.md, summaries/style-extensions.md, summaries/soul-style-agent.md, summaries/report-template.md, summaries/0010-voice-quarto-typst-reporting.md, summaries/0007-style-modular-report-sections-via-quarto-includes.md, summaries/0005-style-quarto-custom-format-extensions-for-report-variants.md, summaries/report_body.md, summaries/NP-20240415-001_report.md, summaries/neuropsych-narrative-writer.md, summaries/neuropsych-data-extractor.md, summaries/conversation-export.md, summaries/FIX_EXPLANATION.md, summaries/index.md, summaries/brainworkup-brand-voice-guide.md, summaries/report-generation.md, summaries/mcp-integration.md, summaries/template-system.md, summaries/quarto-extensions.md, summaries/overview.md, summaries/003-modular-template-structure.md, summaries/002-mcp-llm-integration.md, summaries/001-choose-quarto-typst.md, summaries/SKILL.md, summaries/README.md, summaries/AGENTS_luria.md, summaries/README_luria.md, summaries/deepagents_merged_mem_notes.md, summaries/SETUP_SUMMARY.md, summaries/SESSION_SUMMARY_2025-04-28.md, summaries/RECOVERY_NOTES.md, summaries/PROJECT_SETUP_COMPLETE.md]
brief: End-to-end pipeline for producing structured clinical neuropsychological assessment reports using AI-assisted tools.
---

# Neuropsychological Reporting

Neuropsychological reporting refers to the systematic process of collecting, analyzing, and presenting cognitive and behavioral assessment data in structured clinical documents. In software contexts, it encompasses the tools, pipelines, and templates used to automate or assist in producing these reports from raw assessment data.

The **Luria** toolkit (see [[summaries/README_luria]] and [[summaries/PROJECT_SETUP_COMPLETE]]) is the primary platform implementing this concept — a comprehensive open-source toolkit (MIT License, Python 3.10+) built by BrainWorkup Neuropsychology for end-to-end neuropsychological data processing, statistical analysis, and clinical report generation. The **`cingulate` R package** (see [[summaries/CLAUDE]]) is the R-native implementation of the same pipeline concept, turning neuropsychological test data into publication-quality PDF reports through an R6-first architecture. The **Cingulate Agent Team** (see [[summaries/agent-team]] and [[summaries/2026-04-26-cingulate-agent-team-design]]) is a complementary multi-stage AI pipeline that drives a single patient case from raw CSVs through to a draft PDF and QA issue list, coordinating a set of specialized Claude subagents via the `cingulate-orchestrator`. The **Luria App** redesign (see [[summaries/redesign_20260623110817]]) represents the full-featured clinical desktop interface that unifies all of these components into a practitioner-facing workflow.

## Core Concepts

### What It Involves
- Aggregating scores from standardized cognitive tests (e.g., memory, attention, executive function)
- Mapping raw scores to normative data and clinical interpretations
- Generating structured narrative and tabular reports for clinicians
- Supporting multiple output formats (PDF, HTML, Word, Typst typesetting, [[concepts/quarto]])
- Providing AI-assisted clinical reasoning grounded in patient-specific evidence

### Clinical Domains Typically Covered
- Intellectual functioning (IQ)
- Memory and learning
- Attention and processing speed
- Executive functioning
- Language and visuospatial abilities
- Emotional and behavioral functioning (ADHD, Emotion)

## The Luria App — Redesign (2026)

The Luria App redesign (documented in [[summaries/redesign_20260623110817]]) presents a comprehensive practitioner-facing interface that unifies intake, document management, cognitive assessment, AI-assisted reasoning, and report writing into a single desktop (and mobile companion) application.

### Application Workspace Structure

Luria is organized into a consistent sidebar navigation with three sections:

- **Home:** Report Workspace (`⌘1`)
- **Workspace:** Patient Intake (`⌘2`), Clinical Office (`⌘3`), Console & Synthesis (`⌘4`)
- **Tools:** Visuomotor Sketchpad (`⌘5`), Cognitive Map (`⌘6`), Report Builder (`⌘7`)

Each session is tied to an **Active Case-File** with patient metadata (age, handedness, test count, model version). The interface supports Light, Dark, and Amber color themes.

### Five-Step Progress Workflow

The clinician-facing workflow follows a linear five-step pipeline:

1. **Intake** — Structured history capture with AI auto-tagging
2. **Documents** — Source record ingestion, indexing, and RAG-powered fact extraction
3. **Cognitive Map** — Domain constellation visualization (13 domains, colored by severity)
4. **Synthesis** — AI-assisted pattern detection and diagnostic reasoning
5. **Report** — Section-by-section drafting with voice/tone controls

### Key Interface Pages

#### Report Workspace (Page 01)
The dashboard hub showing all active dossiers, patient header, the progress tracker, and live neurocognitive/neurobehavioral score tables. A **Synthesis Panel** surfaces AI-generated pattern impressions in real time. Mobile companion view mirrors domain severity via color-coded bars.

#### Patient Intake (Page 02)
Structured history capture with auto-tagging via the **Λ (Lambda) AI assistant**. Referral text is parsed into discrete clinical concerns. Developmental and medical history is organized in a flagged table. The AI assistant identifies gaps (e.g., missing prior psychoeducational testing) and suggests follow-up items. This aligns with [[concepts/neurodevelopmental-clinical-intake]] and [[concepts/staged-clinical-intake]] patterns.

#### Clinical Office / Documents (Page 04)
Indexed source documents with RAG-powered fact extraction (24 chunks embedded). Each extracted fact is linked to its source page. Status indicators: `INDEXED`, `PROCESSING`, `NOT ON FILE`. Facts pre-fill the report background section. Document ingestion uses **markitdown** for PDF → Markdown conversion.

#### Console & Synthesis (Page 05)
The core AI reasoning interface — described as "a clinical copilot you argue with." The clinician poses diagnostic questions; **Λ Luria** responds with grounded multi-source reasoning, numbered citations mapped to an Evidence Rail. Actions: `[Add to Synthesis]`, `[Draft this section]`, `[Show reasoning]`. This is the primary surface for [[concepts/clinical-ai-reasoning]] and [[concepts/neuropsychological-synthesis]].

Example exchange:
- *Question:* Is inattention primary ADHD or secondary to dysgraphia/frustration?
- *Luria's answer:* Primary ADHD, citing cross-setting Conners-4 elevations (T=72/70), dissociation from output scores (NEPSY-II SS 76), and onset history predating writing demands — supporting **ADHD-Inattentive (primary), with co-occurring dysgraphia**.

#### Cognitive Map — Amber Theme (Page 07)
A constellation visualization of 13 cognitive domains, node size reflecting test breadth and color encoding severity. The Amber theme is designed as a "dark, single-purpose room" for profile reading at a glance. AI auto-detects co-varying clusters and names the clinical pattern:
> **Frontal-graphomotor cluster** — Attention, executive control, and motor output co-vary and are jointly depressed while verbal reasoning is spared — the ADHD-Inattentive + dysgraphia signature.

This connects to [[concepts/cognitive-domains]], [[concepts/adhd-clinical-features]], [[concepts/dysgraphia]], and [[concepts/executive-function-deficits]].

#### Report Builder (Page 08)
Section-by-section report drafting with AI assistance. Controls include voice tone (Clinical / Balanced / Parent), reading level slider (Grade 8 ↔ Professional), and insertable elements (score table, domain figure, DSM-5 criteria). The AI drafts from ingested tests and source documents with citations active.

### LLM Infrastructure & Local Inference (Page 09)

The Luria App's AI layer runs entirely on local models by default — a privacy-first architecture critical for PHI protection.

**Architecture components:**
- UI components: `IntakeDossier.tsx`, `ReportJobStatus`, `ConsoleChat`
- Orchestration: `redactPhi()` PHI guard, `reportJobs.ts`, `llmAbortContext` (AsyncLocalStorage for mid-fallback cancellation)
- Agents: `agentRunner.ts` → section agents (`nseCodSummary`, `ROCFT`, report-section agents)
- Client: `LocalFallbackLLMClient` with `pickProvider()`

**Fallback chain (priority order):**
1. **oMLX** — local OpenAI-compatible API (PHI-safe)
2. **vMLX** — local Responses API (PHI-safe)
3. **Ollama** — local native API (PHI-safe)
4. **Cloud** — remote fallback, non-PHI only (blocked by `restrictToPreferredProviders: true`)

This implements the [[concepts/fallback-strategy]] and [[concepts/llm-provider-abstraction]] patterns. See [[concepts/local-llm-inference]] and [[concepts/phi-data-handling]] for related concepts. The `restrictToPreferredProviders: true` gate ensures all patient data stays on-device.

**Three data flow paths:**
- A: Intake → encrypted SQLite → `redactPhi()` → local LLM → Clinical Summary
- B: UI → report job → `runReportJob()` → section agents → fallback chain
- C: AgentContext → section agent → LLM inference → structured report section

### Knowledge Base Integration — OpenKB (Page 10)

A proposed (not yet implemented) backend feature: integrating OpenKB as the persistent knowledge layer for the Luria App. OpenKB compiles documents into a persistent, interlinked wiki using LLMs and PageIndex for vectorless long-document retrieval.

**Key distinction from traditional RAG:** Knowledge is compiled once and accumulated — cross-references and contradictions are pre-resolved — rather than re-derived on every query. This connects to [[concepts/knowledge-base-architecture]], [[concepts/knowledge-continuity]], and [[concepts/retrieval-augmented-generation]].

**OpenKB generators:** `openkb query` (grounded Q&A), `openkb chat` (multi-turn), `openkb skill new` (compiles wiki subset into an Anthropic Skill installable by Claude Code, Codex CLI, Gemini CLI, Cursor). Quality gates include `openkb skill validate`, `openkb skill eval`, and `openkb skill history`/`rollback`.

**Tech stack:** markitdown, OpenAI Agents SDK, LiteLLM, Click, watchdog.

See [[concepts/knowledge-base-architecture]] and [[summaries/KNOWLEDGE_BASE_EXPLAINED]] for further treatment.

## The `cingulate` R Package

The `cingulate` R package (documented in [[summaries/CLAUDE]]) is an R6-first system that implements the core CSV→PDF pipeline natively in R, with a small Python sidecar for LLM and PDF tasks. Its pipeline is:

```
CSV (data-raw/csv/) → DuckDB/Parquet → R6 domain processors → per-domain QMD includes → Quarto/Typst → PDF
```

It is a proper package — not a script collection — and must be loaded via `devtools::load_all('.')` rather than `source()`, since R6 generators register on package load.

### Key R6 Classes

- **`WorkflowRunnerR6`** — top-level orchestrator for the full CSV→PDF pipeline
- **`DomainProcessorR6` + `DomainProcessorFactoryR6`** — one processor per cognitive domain; the factory auto-detects which domains exist in input data
- **`NeuropsychResultsR6`** — converts processed scores into clinical narrative text
- **`TableGTR6` / `DotplotR6` / `DrilldownR6`** — rendering components (gt tables, dot plots, Highcharts drilldown)
- **`DuckDBProcessorR6`** — DuckDB-backed staging for performance (CSV → Parquet → in-memory queries via `query_neuropsych()`)
- **`OllamaModelRouterR6` / `ReportLLMBridgeR6` / `DomainPrompterR6`** — LLM routing for narrative generation; models selected per task and per performance mode (`development` / `balanced` / `production`) from `inst/config/ollama_models.yml`
- **`ConfigManagerR6`, `ScoreTypeCacheR6`, `ErrorHandlerR6`, `ExecutionTrackerR6`, `TemplateContentManagerR6`, `PackageManagerR6`** — supporting infrastructure

See [[concepts/r6-class-architecture]] for the cross-document treatment of this design pattern.

### Domain QMD Generation Pattern

Reports are assembled from numbered per-domain Quarto includes:

1. **Text first**: `generate_domain_text_qmd()` → `_02-XX_domain_text.qmd` (narrative)
2. **QMD shell second**: `_02-XX_domain.qmd` includes the text file and adds tables/plots
3. **Multi-rater domains** (emotion, ADHD): separate per-rater files (self/parent/teacher); always call `check_rater_data_exists()` / `check_domain_raters()` before generating; child vs adult reports diverge here
4. **Stable numbering**: `01_iq`, `02_academics`, `03_verbal`, etc. — do not renumber; rendering order and `_quarto.yml` rely on it

The system uses **two-stage rendering with edit protection**: manually edited `_02-XX_<domain>_text.qmd` files can be protected by adding `# manual-edit` as the first line. The interpretation stage skips such files rather than overwriting them — an instance of the [[concepts/edit-protection-pattern]] applied within the agent pipeline.

### Score-Type Handling

Footnotes and z-score conversions branch on whether a row is `t_score`, `scaled_score`, or `standard_score`. `ScoreTypeCacheR6` caches this detection. New tests must have recognized score types — see `R/score_type_utils.R` and `R/neuropsych_test_scoring.R`.

### Key Pitfalls

| Pitfall | Detail |
|---|---|
| **JEP 66 handshake timeout** | Never auto-load `library(cingulate)` in `.Rprofile`; Positron will time out on 40+ heavy imports |
| **`source()` won't work** | R6 generators register on package load; use `devtools::load_all('.')` |
| **Input path ambiguity** | Some helpers accept paths with/without `data/` prefix; `data-raw/csv/` is canonical but gitignored |
| **Internal scales data** | In `R/sysdata.rda`; use `load_scales_internal()` / `safe_use_data_internal()` |
| **LLM model availability** | Verify with `check_available_models()`; use `mode = "development"` for iteration |

## The Luria Platform

Named in honor of A.R. Luria, a foundational figure in neuropsychology, the Luria toolkit provides:

- **Data Processing**: Ingests CSV, Excel, JSON, Parquet, and SQLite data; cleans and validates neuropsychology test scores.
- **Statistical Analysis**: Descriptive stats, t-tests, ANOVA, correlation, regression, normative comparisons, and cognitive domain analysis.
- **Report Generation**: Produces professional reports in HTML, PDF, DOCX, and Markdown using customizable templates (Adult, Pediatric, Forensic, Research).
- **R Integration**: Seamless Python–R interop via `reticulate` and the `cingulate` R package for advanced statistical workflows. See [[concepts/r-python-integration]].
- **CLI & Python API**: Both command-line (`luria init`, `luria process`, `luria analyze`, `luria report`) and programmatic interfaces.
- **Privacy-First Design**: Local processing by default; optional cloud LLM integration (Anthropic, OpenAI) for advanced analysis. See [[concepts/privacy-first-software]].
- **Visualization**: Score profiles, domain heatmaps, longitudinal plots, and normative comparison charts.

### Data Model

Expected tabular structure per row:

| Field | Description |
|---|---|
| `subject_id` | Participant identifier |
| `test_name` | Name of neuropsychological test (e.g., WAIS-IV_VCI) |
| `domain` | Cognitive domain (e.g., memory, attention, language) |
| `raw_score` | Raw test score |
| `scaled_score` | Scaled score |
| `t_score` | T-score |
| `percentile` | Percentile rank |

### Python Package Structure

Following the setup described in [[summaries/SETUP_SUMMARY]], the Luria codebase follows the `src/` layout pattern for Python packaging (see [[concepts/python-project-structure]]):

```
src/luria/
├── core/           # Configuration, logging, utilities
├── data/           # Data loading, cleaning, validation
├── analysis/       # Statistical analysis, visualization
├── reporting/      # Report generation, templates
└── cli.py          # Unified command-line interface
```

R integration lives under `R/` (with the `cingulate` package and `config.R`), tests under `tests/`, documentation under `docs/`, and example notebooks under `examples/` and `notebooks/`.

## The Cingulate Agent Team

The Cingulate Agent Team (see [[summaries/agent-team]]) is a five-stage multi-agent pipeline that drives a single patient case from raw assessment CSVs to a draft PDF report and QA issue list. It is coordinated by the `cingulate-orchestrator` and dispatches five specialized subagents:

```
intake → scoring → interpretation → report → qa
```

| Subagent | Stage | Model | Definition |
|----------|-------|-------|------------|
| `cingulate-orchestrator` | (driver) | opus | `.claude/agents/cingulate-orchestrator.md` |
| `cingulate-intake` | 1 | sonnet | `.claude/agents/cingulate-intake.md` |
| `cingulate-scoring` | 2 | sonnet | `.claude/agents/cingulate-scoring.md` |
| `cingulate-interpretation` | 3 | opus | `.claude/agents/cingulate-interpretation.md` |
| `cingulate-report-writer` | 4 | sonnet | `.claude/agents/cingulate-report-writer.md` |
| `cingulate-quality-reviewer` | 5 | opus | `.claude/agents/cingulate-quality-reviewer.md` |

All artifacts land under `output/<patient_slug>/`. The orchestrator uses `state.json` as its single source of truth for stage progress — aligning with the [[concepts/agent-pipeline-state-management]] pattern.

### Cingulate Stage Status Model

Each stage returns one of four statuses:
- `DONE` — completed successfully
- `DONE_WITH_CONCERNS` — completed with warnings
- `NEEDS_CONTEXT` — missing input
- `BLOCKED` — hard failure; orchestrator halts and surfaces the failing stage, a one-line reason, and the absolute path to the stage log

### Resuming After Failure

To resume after a fix, re-dispatch the orchestrator with the same `patient_slug`. It reads `state.json`, finds the first non-`done` stage, and continues. Stage statuses can be manually overridden in `state.json` — setting a stage to `pending` forces a rerun; setting it to `done` skips it (use cautiously).

### Common Failure Modes

| Symptom | Likely Cause | Fix |
|---------|--------------|-----|
| `BLOCKED` at scoring: "JEP 66 handshake failed" | R startup timeout | `.rs.restartR()`, avoid auto-loading `library(cingulate)` |
| `BLOCKED` at interpretation: "model not available" | Ollama not running | `bash setup_ollama.sh` |
| `BLOCKED` at report: "extension not found" | Missing Quarto extension | `quarto add` or symlink extensions |
| `BLOCKED` at qa: "pdftotext not found" | Poppler missing | `brew install poppler` |
| `DONE_WITH_CONCERNS` at interpretation | LLM empty/garbled output | Re-dispatch with failed domain list, switch `llm_mode` |

### Final Outputs

- **PDF report**: `output/<slug>/report/<slug>.pdf`
- **QA issue list**: `output/<slug>/qa/issue_list.md` (severities: `blocker | major | minor | nit`)

Blockers must be resolved before sign-off. **Human approval is always required** — the agent team never auto-approves a report. This reflects the broader principle of [[concepts/report-review-qa]].

## Report Generation Stack

Both the Luria platform, the `cingulate` R package, and the Cingulate Agent Team use **Quarto** as the document generation framework with **Typst** as the typesetting engine for final PDF output. This decision was first formalized in ADR 001 (see [[summaries/001-choose-quarto-typst]]) and subsequently consolidated in ADR 0010 (see [[summaries/0010-voice-quarto-typst-reporting]]), which supersedes two prior overlapping ADRs and captures the canonical rationale in a single place.

### Why Quarto + Typst

**Quarto** (see [[concepts/quarto]]) was selected because:
- It builds on Pandoc with enhanced features for scientific and technical writing
- It provides native R code chunk support via `knitr`
- It supports multiple output formats: PDF, HTML, Word
- It integrates deeply with the R ecosystem (ggplot2, dplyr) — see [[concepts/r-python-integration]]
- Its YAML-based configuration enables reproducible builds
- It has active community and long-term support

**Typst** (see [[concepts/typst-typesetting]]) was selected over LaTeX because:
- It offers significantly faster, single-pass compilation vs. LaTeX's multi-pass model
- Its rule/function model is more approachable than LaTeX macro-heavy templating
- It uses system fonts directly, simplifying font configuration
- It provides native Unicode support and cleaner error messages
- It integrates with Quarto via `template-partials` (`typst-template.typ`, `typst-show.typ`) as used by the neurotyp extensions

Alternatives considered and rejected:

| Alternative | Reason Rejected |
|---|---|
| LaTeX | Steep learning curve, slow multi-pass compilation, complex errors |
| Pandoc alone | Less structured, harder to maintain complex templates |
| Word templates | Not reproducible, no code execution, poor version control |

**Implementation details:**
- Quarto extensions created at `style/_extensions/brainworkup/` and also at `inst/quarto/_extensions/brainworkup/` (cingulate package)
- Typst templates implemented for pediatric, adult, and forensic report types
- Project configured via `style/templates/typst-report/_quarto.yml`
- Variable substitution system using `_variables.yml` (see [[concepts/yaml-configuration]])
- Custom styling implemented via Typst templates and Quarto format extensions

## Report Templates

### Cingulate Package Templates

In the `cingulate` R package (see [[summaries/CLAUDE]]):
- `_quarto.yml` renders `template.qmd` using the **`neurotyp-pediatric-typst`** format from the project-local extension at `inst/quarto/_extensions/brainworkup/`
- Adult/forensic variants live alongside under `inst/quarto/`
- The `inst/rmarkdown/` skeletons are legacy/secondary
- Two competing main entry points exist (`Cingulate2MainR6.R` and `cingulateMainR6`) — verify which the current workflow runner uses before adding code

### Voice Style System Templates

The **Voice Style** system (documented in [[summaries/overview]]) is the modular report generation framework that implements the Quarto + Typst stack at the file-system level. Its directory structure is:

```text
voice/
├── style/
│   ├── _extensions/
│   │   └── brainworkup/
│   │       ├── neurotyp-adult/
│   │       ├── neurotyp-forensic/
│   │       └── neurotyp-pediatric/
│   └── templates/
│       └── typst-report/
├── config.yml
├── _quarto.yml
├── _variables.yml
└── soul/
```

## System Requirements

The full report generation pipeline requires:

- **Quarto** >=1.4.0
- **R** >=4.0 (with neuro2, dplyr, ggplot2, knitr, etc.)
- **Typst** (latest)
- **Ollama** (local LLM backend)
- **Python** >=3.10 (processing scripts)
- **pdftotext** / Poppler (QA stage, for the Cingulate pipeline)
- **uv** (Python package management)

Configuration is managed through `config.yml`, `_variables.yml`, and a selected Quarto extension (pediatric/adult/forensic).

## Quarto Extensions

Three custom Quarto format extensions (documented in [[summaries/quarto-extensions]] and organized under `style/_extensions/brainworkup/`) serve distinct clinical populations. Each extension is defined by three files:

- **`_extension.yml`**: Declares metadata and registers Typst template partials.
- **`typst-template.typ`**: Defines page geometry, margins, headers/footers, fonts, and section styling.
- **`typst-show.typ`**: Defines Typst show rules for headings, paragraph spacing, lists, tables, figures, block quotes, and code blocks.

### Extension Specifications

| Extension | Audience | Font | Paper | Font Size | Heading Font |
|---|---|---|---|---|---|
| `neurotyp-pediatric` | Children (<18) | Equity B | A4 | 11.5pt | Source Sans 3 |
| `neurotyp-adult` | Adults (18+) | IBM Plex Serif | US Letter | 11.5pt | IBM Plex Sans |
| `neurotyp-forensic` | Forensic/legal | TeX Gyre Termes | US Letter | 12pt | IBM Plex Sans |

All three use APA citation style and do not number sections. See [[concepts/quarto-extensions]] for cross-document treatment.

### Clinical Use Cases

- **Pediatric**: Developmental norms, age-appropriate standards, educational implications, family-focused recommendations.
- **Adult**: Work assessments, disability evaluations, cognitive aging, neurological conditions.
- **Forensic**: Legal standards adherence, detailed methodology documentation, comprehensive disclaimer sections, expert witness testimony preparation. See [[concepts/forensic-neuropsychological-evaluation]].

## Modular Template Architecture

ADR 003 (see [[summaries/003-modular-template-structure]]) formalizes the **modular template system** used to assemble neuropsychological reports from discrete section files. See [[concepts/modular-report-architecture]] for a cross-document treatment of this pattern.

### Structure

- A main `template.qmd` acts as the **orchestrator**
- Individual section files are included via Quarto's `{{< include >}}` mechanism
- A **numbered prefix system** encodes section ordering and hierarchy
- Dynamic domain inclusion via `{{< include _domains_to_include.qmd >}}`
- Shared variables centralized in `_variables.yml`

### Numbered Prefix System

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

First two digits = major section; last two digits = subsection ordering. New sections can be inserted without full renumbering. The `_domains_to_include.qmd` dispatcher at position `02-XX` conditionally includes only the cognitive domain sections relevant to each patient case.

## In the Luria Project

As of the 2025-04-28 architecture refactor (see [[summaries/SESSION_SUMMARY_2025-04-28]]), the system has moved away from a CLI-based orchestration model toward a skill-based multi-agent architecture. The [[summaries/deepagents_merged_mem_notes]] document (May 2026) represents the most detailed and corrected orchestration plan to date, grounding the system in what actually exists on disk and replacing aspirational components with verified tooling.

### Four-Stage LangGraph Pipeline (Core)

At the heart of the Luria architecture is a four-stage LangGraph pipeline that converts raw neuropsychological PDFs into narrative reports (see [[summaries/README_luria]]):

1. **`[parse]`** — PDF ingestion via the PDF Ingestion Parser subagent
2. **`[extract]`** — Structured JSON data extraction via the Neuropsych Data Extractor subagent
3. **`[index]`** — Score indexing via the Sheets Data Indexer subagent
4. **`[report]`** — Narrative report generation via the Narrative Report Generator subagent

The `IngestGraph` state flow is: `START → parse → extract → index → report → END`.

### Stage 1: The PDF Ingestion & Parser Worker

The first stage is the **PDF Ingestion & Parser Worker** — the entry point for all new documents. It fetches, classifies, cleans, and structures raw PDF content before any downstream analysis occurs.

**Key Design Principles:**
- **Fidelity first**: No interpretation at this stage; scores and structure reproduced verbatim.
- **PHI safety**: All protected health information anonymized before output (see [[concepts/phi-data-handling]]).
- **Fail loudly**: Inaccessible documents produce explicit error reports rather than silent failures.

### Stage 2: The Neuropsychological Data Extractor

Receives cleaned text and performs deep structured extraction of all clinically relevant data, outputting a JSON array (one row per test or subtest).

**Extraction schema includes:** `doc_id`, `doc_type`, `date`, `test_name`, `subtest_name`, `raw_score`, `scaled_score`, `standard_score`, `t_score`, `percentile_rank`, `classification`, `composite_index`, `primary_diagnosis`, `clinical_impression`, `cognitive_domains_affected`, `recommendations`, `notable_findings`.

### Stage 4: The Narrative Report Generator

Organizes JSON records by cognitive domain, interprets each score using normative benchmarks, writes a full clinician-quality narrative report, and persists output to `data/reports/{doc_id}/{doc_id}_report.md`.

Report generation follows the standardized six-section shape defined in the `luria-report-writing` skill:

1. **Reason for Referral**
2. **Sources Reviewed**
3. **Behavioral Observations**
4. **Test Results by Domain**
5. **Summary and Impressions**
6. **Recommendations**

#### Normative Benchmarks

| Score Type | Mean | SD |
|---|---|---|
| Scaled scores | 10 | 3 |
| Standard scores | 100 | 15 |
| T-scores | 50 | 10 |

**Percentile Classifications:**
- >75th → Above Average
- 25th–75th → Average
- 9th–24th → Low Average
- 2nd–8th → Borderline
- <2nd → Impaired

#### Report Writing Rules

- **Professional clinical prose**: All narrative sections written in formal, clinician-appropriate language.
- **Preserve redaction tokens**: All PHI anonymization tokens (`[PATIENT_ID]`, `[CLINICIAN]`) must be preserved as-is throughout the report. See [[concepts/redaction-tokens]].
- **No raw scores in prose when tables carry them**: Prose interprets; tables present.
- **No external cloud documents**: Output stays entirely within Luria's report surfaces.

### Five-Phase Clinical Pipeline

The corrected architecture (documented in [[summaries/deepagents_merged_mem_notes]]) defines a five-phase pipeline that mirrors the clinical visit structure:

```
Phase A — Intake (Visit 1 / NSE)
Phase B — Testing (Visit 2 / NT)
Phase C — SIRF: Summary, Impression, Recommendations (Visit 3)
Phase D — Final Assembly + Quality Review
Phase E — Render (deterministic, no LLM)
```

#### Phase A — Intake

Sequential with one parallel window (A2/A3/A4 after A1). A5 (`nse_cod_summary`) is a critical PII redaction gate: it runs Presidio (`pii_presidio.py`) to detect and mask names, DOB, addresses, phone numbers, MRN, and SSN. All downstream agents receive only redacted content. See [[concepts/pii-redaction-pipelines]] for related patterns.

#### Phase B — Testing

The pipeline operates on **9 confirmed domains** — 7 neurocognitive (IQ, Academics, Verbal, Spatial, Memory, Executive, Motor) and 2 neurobehavioral (ADHD, Emotion). Per-domain pipeline:

```
DataPrepAgent
  └── filters/classifies → data.json
       ├── TextSubAgent → _02-XX_domain_text.qmd
       ├── TableSubAgent → TableGTR6.R → table PNG
       └── FigureSubAgent → DotplotR6.R → figures
```

Max concurrency is ~21 (7 domains × 3 parallel Text/Table/Figure agents). Each domain agent returns one of `COMPLETED`, `DEGRADED`, `SKIPPED`, or `FAILED` (retryable/non-retryable). Retryable failures get up to 2 retries with backoff.

#### Phase C — SIRF

Sequential. Each step depends on all previous phases: `sirf_summary` (domain + whole-profile interpretation), `sirf_impression` (DSM-5/ICD-10 diagnosis), `sirf_recs` (recommendations with RAG). C3 uses PageIndex + local `rag_db/` for evidence-based recommendations (see [[concepts/retrieval-augmented-generation]]).

#### Phase D — Final Assembly

- **D1** `luria-neuropsych-orchestrator` — assembles all phase outputs into a full QMD following the modular template structure.
- **D2** `luria-quality-review` — checks completeness, PHI leaks (Presidio re-scan), score↔narrative consistency, AACN classification terminology, validity language, test security, and DSM-5/ICD-10 code accuracy. Max 3 correction cycles before human escalation.

#### Phase E — Render

Deterministic, no LLM. Canonical Quarto templates and the voice/brand sync bridge (`voice_quarto.py`) drive final output generation:
```
quarto render Biggie.qmd --to neurotyp-pediatric-typst → Biggie.pdf
```

### Orchestration Runtime

The corrected plan uses **LangGraph** (`neuropsych_agent/graph.py`) as the orchestration runtime. See [[concepts/langgraph-agent-workflows]] and [[concepts/multi-agent-orchestration]] for related patterns.

### Current Skill Architecture (Post-Refactor)

The system uses a `luria-neuropsych-orchestrator` skill that delegates to sub-skills:

```
luria-neuropsych-orchestrator
├── luria-case-intake          → Patient data normalization
├── luria-score-processing     → Test score extraction
├── luria-interpretation       → Domain-level analysis
├── luria-report-writing       → Report generation
└── luria-quality-review       → Final validation
```

### Model Routing Summary

| Role | Model | Notes |
|---|---|---|
| Orchestrator / patient-facing output | MLX-Qwen3.5-35B-A3B reasoning distilled | Best quality |
| Summary / diagnosis / review | Qwen3.6-35B-A3B-oQ4 | Strong reasoning |
| Domain narrative (Text subagent) | gpt-oss-20b-MXFP4-Q8 | Clinical prose |
| Fast / deterministic / DataPrep | granite-4.1-8b-nvfp4 | Temp=0 |
| Embeddings | nomicai-modernbert-embed-base-bf16 | 300M, soul agent |

All models served via the local oMLX endpoint at `http://127.0.0.1:8000`. See [[concepts/local-llm-inference]] and [[concepts/omlx-server]] for related patterns.

### Key File Locations

| Component | Path |
|---|---|
| LangGraph graph | `neuropsych_agent/graph.py` |
| PII redaction tool | `neuropsych_agent/tools/pii_presidio.py` |
| R Domain Processors | `agent/cingulate/R/DomainProcessorR6.R` |
| Cingulate package entry points | `R/workflow_*.R` |
| Cingulate package config | `inst/config/ollama_models.yml` |
| Cingulate Quarto extension | `inst/quarto/_extensions/brainworkup/` |
| PageIndex Service | `rag/page-index/app/service.py` |
| Report output | `data/reports/{doc_id}/{doc_id}_report.md` |
| Quarto template (main orchestrator) | `template.qmd` (root) |
| Quarto extensions | `style/_extensions/brainworkup/` |
| Typst report templates | `style/templates/typst-report/` |
| Variable substitution | `_variables.yml` |
| Cingulate agent team runbook | `.claude/agents/cingulate-*.md` |
| Cingulate output directory | `output/<patient_slug>/` |
| Cingulate state file | `output/<patient_slug>/state.json` |

## Quality Assurance

### Pre-Generation Checks
1. Verify PDF data is complete
2. Check patient information accuracy
3. Confirm test battery matches assessment
4. Validate lookup table entries

### Post-Generation Checks
1. Review PDF for formatting issues
2. Verify all sections included
3. Check table/figure rendering
4. Validate diagnostic codes
5. Confirm signature block
6. Review recommendations completeness

### Cingulate QA Stage

The `cingulate-quality-reviewer` subagent (Stage 5) generates a QA issue list at `output/<slug>/qa/issue_list.md` with four severity levels: `blocker`, `major`, `minor`, and `nit`. All blockers must be resolved before any human sign-off. This integrates with the broader [[concepts/report-review-qa]] concept.

### Verification Infrastructure

A smoke test script (`scripts/smoke_test_paths.py`) validates all static paths the pipeline depends on, AST-parses every `.py` file, and confirms the `WORKSPACE_ROOT` resolution math in `nodes.py`:

```bash
python3 scripts/smoke_test_paths.py
# Expected: PASS — all required paths resolve and all .py files parse.
```

The Cingulate pipeline has its own smoke test: `Rscript -e "devtools::load_all('.')); cingulate_llm_smoke_test()"` validates the LLM/Ollama integration before any real PHI run (see [[concepts/smoke-test-scripts]]).

## Relation to Other Concepts

### Document Generation Stack
The Quarto + Typst stack is the foundation of all final report rendering, formalized across two complementary ADRs: ADR 001 (see [[summaries/001-choose-quarto-typst]]) establishing the initial decision and ADR 0010 (see [[summaries/0010-voice-quarto-typst-reporting]]) consolidating the canonical rationale. The modular template system (formalized in [[summaries/003-modular-template-structure]]) organizes assembled content into ordered, reusable section files. See [[concepts/quarto]] and [[concepts/typst-typesetting]] for deeper treatment, and [[concepts/modular-report-architecture]] for cross-document synthesis of the modular design pattern.

### Luria App Interface Layer
The redesigned Luria App (see [[summaries/redesign_20260623110817]]) provides the practitioner-facing surface above the reporting pipeline: intake forms, document ingestion, cognitive mapping, AI-assisted synthesis, and report drafting. It connects to [[concepts/luria-overview]], [[concepts/luria-neuropsych-pipeline]], and [[concepts/clinical-ai-reasoning]].

### Natural Language Generation
Modern neuropsychological reporting increasingly uses language models to generate narrative interpretations from structured score data. This overlaps with [[concepts/clinical-nlp-pipelines]], where NLP tools process and generate clinical text, and [[concepts/clinical-narrative-generation]].

### Privacy and Data Sensitivity
Neuropsychological data is highly sensitive protected health information (PHI). All PHI is anonymized at ingestion using explicit tokens (`[PATIENT_ID]`, `[CLINICIAN]`) and enforced again by the PII redaction gate at Phase A5. The `output/` directory in the Cingulate pipeline is gitignored to protect PHI. The `luria-report-writing` skill further mandates preservation of [[concepts/redaction-tokens]] throughout the final report. See [[concepts/privacy-first-software]] and [[concepts/phi-data-handling]]. See also [[concepts/pii-redaction-pipelines]] and [[concepts/clinical-data-privacy]].

### RAG and Knowledge Retrieval
Report generation is grounded in authoritative clinical sources via [[concepts/retrieval-augmented-generation]]. The `rag/page-index/` service provides document upload, indexing, and chat capabilities.

### Local and Offline Processing
Given PHI sensitivity, all AI-assisted interpretation runs locally via the oMLX endpoint rather than external APIs — aligning with [[concepts/local-llm-inference]] and [[concepts/local-first-architecture]]. The Cingulate pipeline similarly requires Ollama to be running locally with models pulled.

### R and Python Integration
The `cingulate` package uses R6 classes for domain-level processing (norming computations, DotplotR6.R, TableGTR6.R), with Python handling orchestration. See [[concepts/r-python-integration]] for related patterns.

### Multi-Agent Orchestration
The full pipeline coordinates 20+ specialized agents across five phases using LangGraph StateGraph. The Cingulate Agent Team adds a parallel orchestration model using Claude subagents for a single-patient run. See [[concepts/multi-agent-orchestration]] and [[concepts/langgraph-agent-workflows]].

### Subagent Architecture
Both the Luria LangGraph pipeline and the Cingulate Agent Team use the pattern of dedicated subagents per stage, each with a defined scope and status contract. See [[concepts/subagent-architecture]] for cross-document treatment.

### Agent Pipeline State Management
The Cingulate `state.json` and the Luria LangGraph `IngestGraph` state both exemplify the [[concepts/agent-pipeline-state-management]] pattern: a shared, inspectable artifact that records stage progress and enables resumption after failure.

### Documentation and Plain Text
Structured reporting benefits from plain-text and documentation-as-code approaches. See [[concepts/plain-text-documentation]] and [[concepts/documentation-as-code]].

## Design Considerations

| Concern | Approach |
|---|---|
| Data privacy | Local processing, Presidio PII redaction at A5, no cloud upload; `output/` gitignored in Cingulate pipeline |
| Reproducibility | Version-controlled Quarto/Typst templates, subagent prompts, and configs |
| Domain coverage | 9 confirmed domains (7 neurocog + 2 neurobehav); no hallucinated domains |
| Failure isolation | Per-domain COMPLETED/DEGRADED/SKIPPED/FAILED states with retry logic; Cingulate DONE/DONE_WITH_CONCERNS/NEEDS_CONTEXT/BLOCKED |
| Statistical rigor | R6 domain processors for norming; TableGTR6.R and DotplotR6.R |
| Extensibility | Skill-based orchestration, modular Python package structure |
| R session safety | Never auto-load `library(cingulate)` in `.Rprofile`; use `devtools::load_all('.')` |
| Score fidelity | Exact numerical values preserved from ingestion through narrative generation |
| Edit protection | `# manual-edit` header guards interpretation stage from overwriting hand-edited QMD files |
| Report integrity | No fabricated scores; hedging language; PHI anonymized with tokens |
| Human oversight | Agent teams never auto-approve reports; all final sign-off is a human decision |
| Typesetting engine | Typst preferred over LaTeX: faster single-pass compilation, cleaner errors, direct system font access |
| Report types | Pediatric, adult, and forensic templates maintained as Quarto extensions |
| Template modularity | ADR 003 numbered prefix system; new sections insertable without full renumbering |
| Dynamic section dispatch | `_domains_to_include.qmd` conditionally includes only patient-relevant domains |
| Runtime config | Three-file config pattern: `_quarto.yml`, `_variables.yml`, `config.yml` |
| LLM fallback | oMLX → vMLX → Ollama → Cloud (PHI-blocked); `restrictToPreferredProviders: true` enforces local-only for patient data |
| Knowledge accumulation | Proposed OpenKB integration compiles clinical sources into persistent wiki rather than re-deriving on every query |

## Summary

Neuropsychological reporting sits at the intersection of clinical practice, statistical analysis, and software engineering. Four complementary systems implement this concept:

1. The **`cingulate` R package** (see [[summaries/CLAUDE]]) — an R6-first, package-grade implementation of the CSV→PDF pipeline with domain processors, DuckDB staging, Ollama-backed narrative generation, and Quarto/Typst rendering.

2. The **Luria platform** (see [[summaries/PROJECT_SETUP_COMPLETE]], [[summaries/SETUP_SUMMARY]], [[summaries/RECOVERY_NOTES]], [[summaries/SESSION_SUMMARY_2025-04-28]], and [[summaries/deepagents_merged_mem_notes]]) — a comprehensive Python/R toolkit that has evolved from a simple CLI-based pipeline to a skill-based multi-agent architecture with a fully specified five-phase orchestration plan.

3. The **Cingulate Agent Team** (see [[summaries/agent-team]] and [[summaries/2026-04-26-cingulate-agent-team-design]]) — a streamlined, operator-facing Claude subagent pipeline for single-patient runs from raw CSVs to reviewed PDF.

4. The **Luria App** (see [[summaries/redesign_20260623110817]]) — the full-featured practitioner desktop interface providing intake, document ingestion, cognitive mapping, AI-assisted synthesis with grounded citations, and report drafting with voice/tone controls — all running on local LLMs with a PHI-safe fallback chain.

The report generation stack rests on two complementary architectural decisions: **ADR 001** (see [[summaries/001-choose-quarto-typst]]) first selected Quarto + Typst, and **ADR 0010** (see [[summaries/0010-voice-quarto-typst-reporting]]) consolidates this as the canonical choice. **ADR 003** implements a modular template system where a main `template.qmd` orchestrator assembles individual numbered section files via Quarto's `{{< include >}}` mechanism.

The `luria-report-writing` skill (v0.1.0, documented in [[summaries/SKILL]]) is the authoritative codification of how structured findings become polished clinical prose: its six-section shape, insistence on professional clinical prose, protection of [[concepts/redaction-tokens]], prohibition on raw scores in narrative, and restriction to Luria's own report surfaces define the non-negotiable floor for all report output.

See also: [[summaries/002-mcp-llm-integration]] | [[summaries/template-system]] | [[summaries/mcp-integration]] | [[summaries/report-generation]] | [[summaries/brainworkup-brand-voice-guide]] | [[summaries/neuropsych-data-extractor]] | [[summaries/neuropsych-narrative-writer]] | [[summaries/conversation-export]] | [[summaries/FIX_EXPLANATION]] | [[summaries/NP-20240415-001_report]] | [[summaries/report_body]] | [[summaries/0005-style-quarto-custom-format-extensions-for-report-variants]] | [[summaries/0007-style-modular-report-sections-via-quarto-includes]] | [[summaries/soul-style-agent]] | [[summaries/style-extensions]] | [[summaries/style-trainer]] | [[summaries/report-rendering-pipeline]] | [[summaries/style-training-to-report-drafting]] | [[summaries/customization]] | [[summaries/0007-voice-modular-report-sections-via-quarto-includes]] | [[summaries/2026-04-26-cingulate-agent-team-design]] | [[summaries/CLAUDE]]

See also: [[summaries/README]]

See also: [[summaries/LLM_AGENT_MAP]]

See also: [[summaries/neurobehav.prompt]]

See also: [[summaries/nse_narrative]]

See also: [[summaries/nt_interpretation]]

See also: [[summaries/redesign_20260623110817]]

See also: [[summaries/Apply-to-Y-Combinator-JWT]]

See also: [[summaries/Luria_AI_Q4_Investor_Memo_2026]]

See also: [[summaries/cerner-autotext]]