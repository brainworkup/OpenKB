---
sources: [summaries/Luria_AI_Q4_Investor_Memo_2026.md, summaries/Apply-to-Y-Combinator-JWT.md, summaries/redesign_20260623110910.md, summaries/redesign_20260623110817.md, summaries/Introducing-FrontierCode.md, summaries/LLM_AGENT_MAP.md, summaries/CLAUDE.md, summaries/agent-team.md, summaries/DEPENDENCIES.md, summaries/2026-04-26-cingulate-agent-team-design.md, summaries/README.md, summaries/SESSION_SUMMARY_2025-04-28.md, summaries/neuropsych-pdf-parser.md, summaries/neuropsych-narrative-writer.md, summaries/clinical-validity-reviewer.md, summaries/responses_to_claude.md, summaries/SKILL.md, summaries/AGENTS_luria.md, summaries/README_luria.md, summaries/deepagents_merged_mem_notes.md]
brief: Patterns for coordinating specialized agents across structured workflows.
---

# Multi-Agent Orchestration Patterns

Multi-agent orchestration refers to the coordination of multiple specialized AI agents — each with defined roles, tools, context boundaries, and model assignments — across a structured workflow. Rather than relying on a single monolithic agent, orchestration patterns decompose complex tasks into discrete agents that can run sequentially, in parallel, or in nested hierarchies.

In the Luria context, orchestration is not just a software architecture preference; it is the operating model for a local-first neuropsychological evaluation system intended to execute large portions of the workflow autonomously while preserving clinical quality and strict PHI boundaries. The YC application framing of Luria sharpens the core product thesis behind this page: a team of agents and subagents can coordinate intake, data organization, scoring, interpretation, report drafting, and review in a way that mirrors the real structure of neuropsychological work while keeping sensitive data local. That product framing is closely aligned with [[concepts/luria-overview]], [[concepts/neuropsychological-assessment-workflow]], [[concepts/privacy-first-software]], and [[concepts/phi-data-handling]].

See also: [[summaries/deepagents_merged_mem_notes]] for a detailed real-world implementation correcting an aspirational orchestration sketch against a working neuropsychological report pipeline, [[summaries/README_luria]] for a minimal starter kit demonstrating the core patterns in a clean portable form, and [[summaries/Apply-to-Y-Combinator-JWT]] for the founder-facing articulation of why this orchestration model exists.

---

## Core Concepts

### Sequential vs. Parallel Execution

Most pipelines combine both modes:
- **Sequential:** Agent B cannot start until Agent A completes. Used when downstream work depends on upstream outputs (e.g., PII redaction must happen before any subagent sees patient data).
- **Parallel (fan-out):** Multiple agents run simultaneously on independent subtasks, then results are collected (fan-in). Used when subtasks share no data dependencies.

A hybrid example from the Luria pipeline:
```
A1 (intake) → [A2, A3, A4 in parallel] → A5 (join + redact) → A6
```

The Luria LangGraph Starter Kit demonstrates a simpler sequential form:
```
START → parse → extract → index → report → END
```
This pattern — each stage consuming and enriching a shared `PipelineState` TypedDict — is the minimal viable version of the full orchestrated pipeline and a useful starting template for new domains.

The cingulate agent team (see [[summaries/agent-team]] and [[summaries/2026-04-26-cingulate-agent-team-design]]) uses a strict five-stage sequential chain:
```
intake → scoring → interpretation → report → qa
```
Each stage is its own subagent under `.claude/agents/cingulate-*.md`, driven by the `cingulate-orchestrator`. No stage begins until the previous completes and writes its status to `state.json`.

The `cingulate` R package formalizes this further through its `OllamaModelRouterR6` + `ReportLLMBridgeR6` + `DomainProcessorFactoryR6` stack (see [[summaries/LLM_AGENT_MAP]]). The bridge's `run_stage()` method is the atomic unit of orchestration: it calls the router, writes artifacts, logs timing metadata, and caches content-addressed results — all in one call. A `run_pipeline(stages)` method chains these calls sequentially, replicating the linear chain pattern at the R level.

The YC application reinforces why this sequential backbone matters in practice: neuropsychological evaluations are long, high-stakes, and integrative, with report writing as the core deliverable. An orchestration design that cleanly stages collection, transformation, synthesis, and review is a better fit for this domain than a single generalist agent attempting to reason over the entire case in one pass. This is especially true for pediatric and forensic cases, where workflow length and clinical sensitivity increase substantially.

### Fan-Out / Fan-In Pattern

The fan-out pattern spawns N subagents from a single orchestrator node, each handling one slice of the work. Results are collected at a join node before the pipeline continues.

In [[concepts/langgraph-agent-workflows]], this is implemented via:
- **`Send` API (v0.2+):** Each domain is a `Send` to the same graph node with different state. A reducer collects results. Preferred for production — gives per-agent traceability.
- **`asyncio.gather` inside a node:** One node spawns N async calls. Simpler and faster to implement, but loses per-agent observability.

Example from the Luria B3 phase — 7 neurocognitive domain subagents run in parallel:
```
nt_neurocog (orchestrator)
  ├── domain-iq          (DataPrep → Text | Table | Figure)
  ├── domain-academics   (DataPrep → Text | Table | Figure)
  ├── domain-verbal      (DataPrep → Text | Table | Figure)
  ├── domain-spatial     (DataPrep → Text | Table | Figure)
  ├── domain-memory      (DataPrep → Text | Table | Figure)
  ├── domain-executive   (DataPrep → Text | Table | Figure)
  └── domain-motor       (DataPrep → Text | Table | Figure)
```

Max concurrency in this configuration: ~21 agents (7 domains × 3 parallel Text/Table/Figure subagents). Actual average is lower (~12–15) because domains finish at different times.

From the product perspective described in the YC application, this pattern is what makes the claim of an "agent team" concrete rather than rhetorical. The product is not merely using multiple prompts; it is decomposing a real clinical workflow into specialized units whose outputs can later be integrated into a coherent report. That distinction matters because the founder's stated differentiation is not just automation, but automation that preserves cross-domain integration and clinical voice.

### Per-Domain Subgraph Pattern

A reusable subgraph is defined once and instantiated per domain with different inputs. Each instantiation is isolated — state does not leak between domains. This is the "stamp" or "template" pattern:

```
DataPrepAgent → [TextSubAgent | TableSubAgent | FigureSubAgent] → Typst
```

The same graph topology handles all 9 domains (7 neurocognitive + 2 neurobehavioral) in the Luria pipeline, differing only in input parquet files and QMD stub paths.

At the R level, `DomainProcessorFactoryR6` implements the same pattern: a single `create_processor(domain_key, age_group)` call returns a domain-specific processor using the same base class, with differences encapsulated in the factory. Batch creation via `batch_create()` mirrors the Python fan-out, and multi-rater domains (ADHD, emotion) use `create_multi_processor()` to spawn self/parent/teacher variants — a three-way fan-out within a single domain.

This reuse pattern is central to any domain-specific AI system that must maintain consistency across many output sections. In Luria's case, it supports the founder's claim that the system can integrate across neurocognitive and neurobehavioral domains while still respecting domain-specific data structures.

---

## Role-Based Model Routing

A foundational pattern in the `cingulate` stack is decoupling prompt logic from model selection via a YAML-configured role router. The `OllamaModelRouterR6` maps every clinical role to a model ID and fallback, so orchestrators and subagents specify *what kind of reasoning is needed*, not *which model to call*. See [[concepts/role-based-llm-routing]] for the full pattern.

The 13 defined clinical roles cover the full neuropsychological reporting workflow:

| Role | Purpose |
|---|---|
| `summarize_domain` | Summarise a single cognitive domain |
| `interpret_rating_scales` | Interpret behavioural rating scale data |
| `quick_interpret` | Short 1–2 sentence interpretation |
| `integrate_evaluation` | Cross-domain clinical impression |
| `generate_recommendations` | Evidence-based recommendations |
| `differential_diagnosis` | DSM-5 differential considerations |
| `overall_summary` | Executive summary |
| `detailed_interpretation` | Long-form narrative |
| `embeddings` | Generate text embeddings |
| `finalize_diagnostic_summary` | Final diagnostic paragraph |
| `quick_summary` | Brief parent/school summary |
| `generate_narrative` | Full prose narrative |
| `fallback_general` | Generic fallback for any prompt |

This role abstraction insulates the orchestration layer from model changes: swapping a model requires editing `ollama_models.yml` only, not any agent code. The `fallback_general` role ensures the system degrades gracefully when a specialized model is unavailable — a concrete implementation of [[concepts/fallback-strategy]].

The YC application adds strategic context here: if the product promise is a local-first clinical system rather than a generic cloud chatbot, then orchestration and model routing are part of the product boundary. Choosing which model handles cross-domain interpretation versus deterministic scoring support is not a back-end detail; it is a mechanism for preserving privacy, controlling cost, and protecting report quality.

---

## Skill-Based Orchestration (2025 Pattern)

As of the April 2025 architecture refactor (see [[summaries/SESSION_SUMMARY_2025-04-28]]), the Luria system moved from an explicit CLI orchestrator and visit-scoped agents to a **skill-based orchestration pattern**. The `luria-neuropsych-orchestrator` skill now delegates to named sub-skills:

```
luria-neuropsych-orchestrator
├── luria-case-intake          → Patient data normalization
├── luria-score-processing     → Test score extraction
├── luria-interpretation       → Domain-level analysis
├── luria-report-writing       → Report generation
└── luria-quality-review       → Final validation
```

Each skill delegates further to one of three agent layers:
- **Python LangGraph nodes** (PDF parse, extract, index, report) — defined in `app/streamlit/neuropsych_agent/nodes.py` and wired in `graph.py`
- **R6 Domain Processors** (IQ, Memory, Attention, Language, etc.) — implemented in `agent/cingulate/R/DomainProcessorR6.R` and orchestrated via `WorkflowRunnerR6.R`
- **PageIndex service agents** (document upload, chat, export) — served by `rag/page-index/app/service.py`

This pattern replaced the old CSV extraction intake directory, which was migrated entirely into the `cingulate` R package. The Streamlit app is now the single main entry point, exposing a 4-tab interface: Ingest, Reference, Ask, Knowledge.

The refactor also renamed `luria_docling/` to `rag/docling/`, consolidating PII obfuscation under `rag/docling/detect_pii.py`.

The YC application's language of a "team of agents and subagents" maps directly onto this skill decomposition. It provides an external-facing description of the same internal architecture: a system built from reusable workflow components rather than a single opaque model call.

---

## Repo-Bound Skill-to-Subagent Binding (Cingulate Pattern)

The April 2026 cingulate agent team design (see [[summaries/2026-04-26-cingulate-agent-team-design]]) introduces a concrete implementation of skill-based orchestration that binds abstract `luria-*` skills to repo-specific `cingulate-*` subagents stored under `.claude/agents/`. This creates a clean two-layer architecture:

```
luria-* skills (clinical reasoning, ~1 paragraph each)
        ▲ uses
cingulate-* subagents (.claude/agents/, project-local)
        ▲ dispatched by
cingulate-orchestrator (top-level subagent)
```

- **`luria-*` skills** describe *what* each clinical stage does and are intentionally reusable across projects. See [[concepts/luria-skills]] and [[concepts/skills-modules]].
- **`cingulate-*` subagents** bind those stages to a specific repo's R API, file conventions, and Quarto templates — most of the project-specific logic lives here.

The orchestrator reads `state.json`, dispatches the next `pending` stage, and **does not auto-retry**. Failures halt the chain and surface to the human reviewer. This is an explicit design choice for clinical contexts where silent retries could mask data errors.

Subagent roster:

| Subagent | Stage | Model | Responsibility |
|---|---|---|---|
| `cingulate-orchestrator` | (driver) | opus | Drives chain, manages `state.json`, dispatches stages, surfaces failures |
| `cingulate-intake` | 1 | sonnet | Normalize referral, records, interview, NSE; track missing data |
| `cingulate-scoring` | 2 | sonnet | Load CSVs via DuckDB, run domain processors, produce scored tables |
| `cingulate-interpretation` | 3 | opus | Generate per-domain narrative QMD files via LLM router |
| `cingulate-report-writer` | 4 | sonnet | Assemble template, render Quarto/Typst → PDF |
| `cingulate-quality-reviewer` | 5 | opus | PHI scan, completeness, validity language, test-security review |

Each stage subagent receives only its relevant context slice, not the full conversation history — a key decoupling principle for keeping subagent prompts self-contained.

### Launching a Run

The orchestrator is invoked via a natural-language prompt in Claude Code, specifying `patient_slug`, `age_group`, referral question, records, and LLM mode. The orchestrator creates the output directory, initializes `state.json`, dispatches each stage in order, and emits a final summary with the PDF path and QA issue count.

To **resume** after a fix: re-dispatch the orchestrator with the same `patient_slug`; it reads `state.json` and continues from the first non-`done` stage. To **force a rerun** of a stage: set its status back to `pending` in `state.json`. To **skip** a stage: set it to `done` (use cautiously).

At the R level, `cingulate_run_llm_then_render()` provides a single-call equivalent for the LLM generation + Quarto render steps, with an optional `parallel = TRUE` flag to fan out domain processing across CPU cores. The high-level `cingulate_workflow()` function wraps the entire data → domains → LLM → report chain in one call — the R equivalent of re-dispatching the orchestrator from scratch.

### Stage Contracts

Each stage subagent's prompt is a self-contained brief: it gets the workspace path and the slice of context it needs, never the full conversation history. The read/write contract per stage:

| Stage | Reads | Writes | Key API Calls |
|-------|-------|--------|---------------|
| intake | referral text, records, interview/NSE notes | `intake/packet.md`, `intake/missing_data.md` | optional `cingulate_quick_start()` |
| scoring | `data-raw/csv/*` | `duckdb/staged.parquet`, `scoring/<domain>_scored.csv` | `load_data_duckdb()`, `process_all_domains()`, `query_neuropsych()` |
| interpretation | `scoring/*`, `intake/packet.md` | `interpretation/_02-XX_<domain>_text.qmd` | `generate_domain_text_qmd()`, `process_domains_with_llm()` |
| report | `interpretation/*`, `intake/packet.md` | `report/template.qmd`, `report/<slug>.pdf` | `generate_assessment_report()`, `quarto render` |
| qa | `report/<slug>.pdf`, all upstream outputs | `qa/issue_list.md` | `pdftotext` + heuristics; no R |

---

## Per-Patient Workspace as the Inter-Stage Contract

The cingulate design formalizes the **workspace-as-contract** pattern: all stages read and write under a single `output/<patient_slug>/` directory. This is the only inter-stage contract — no shared in-memory state, no direct subagent-to-subagent calls. See [[concepts/per-patient-workspace]] for the full specification.

Key workspace components:
- `state.json` — orchestrator's source of truth; stage statuses: `pending | in_progress | done | error`
- `intake/packet.md` — normalized referral + records + NSE summary
- `intake/missing_data.md` — explicit list of unknowns
- `duckdb/staged.parquet` — scoring stage output
- `scoring/<domain>_scored.csv` — one per active domain
- `interpretation/_02-XX_<domain>_text.qmd` — narrative includes
- `report/<patient_slug>.pdf` — rendered output
- `qa/issue_list.md` — quality reviewer findings
- `logs/<stage>.log` — per-stage logs

The `ReportLLMBridgeR6` artifact directory is the R-level analog: each `run_stage()` call writes `<stage_id>.md`, `<stage_id>.meta.json`, and appends to `run.log.jsonl`. Content-addressed caching under `.cache/<hash>.rds` means re-running an unchanged stage is a no-op — the bridge returns the cached result immediately.

On `error`, the stage writes a one-line `reason` field and a stack trace to `logs/<stage>.log`. The orchestrator surfaces this to the user and halts.

This workspace contract is also what makes a local-first clinical product operationally plausible. If raw artifacts, derived tables, narratives, and QA outputs all live inside a bounded patient workspace on local infrastructure, orchestration can proceed without depending on a remote centralized data plane. That aligns directly with [[concepts/local-first-architecture]], [[concepts/local-llm-inference]], and [[concepts/clinical-data-privacy]].

---

## Inherited Conventions Across Subagents

The cingulate design documents conventions that each subagent inherits independently — restated in each subagent definition rather than centralized, because each runs in its own isolated context:

1. Use `devtools::load_all('.')` — not `library(cingulate)`
2. Default LLM mode is `development`; switch to `production` only when explicitly instructed
3. Never write to `data-raw/csv/`; always copy inputs into the patient workspace first
4. Domain numbering is fixed (`01_iq`, `02_academics`, …) — do not renumber
5. Check for `# manual-edit` marker before regenerating any `_02-XX_*_text.qmd` file (see [[concepts/edit-protection-pattern]])
6. All paths in logs and `state.json` are absolute
7. One git commit per stage on tracked branches: `feat(<patient_slug>): <stage> complete`
8. `patient_slug` format: `lower_snake_case`, max 64 chars, ASCII only

A known CWD constraint: several cingulate helpers expect `setwd(workspace_path)` after `devtools::load_all()`. Stage subagents must do this explicitly; the package does not currently accept a workspace-path argument. Threading one through cleanly is flagged as a future refactor.

---

## Prompt-to-QMD Mapping as Orchestration Contract

The `cingulate` package formalizes a mapping between domain keyword, prompt template file, and target QMD section. This mapping is itself an orchestration contract: the interpretation subagent uses it to determine which prompt file to load and which QMD file to inject results into. Each entry in the table (e.g., `promem` → `inst/prompts/promem.qmd` → `_02-05_memory_text.qmd`) defines one unit of the interpretation stage's work.

`NeuropsychResultsR6$process(llm=TRUE, domain_keyword="promem")` is the atomic call: it reads the prompt file, calls `generate_domain_summary_from_master()` via the router, and injects a `<summary>…</summary>` block into the QMD using `LLM_CONTEXT_START…END` markers. The content-addressed hash means this block survives re-runs unchanged unless the underlying data changes — a fine-grained version of the bridge's caching at the QMD level.

Special-purpose keywords extend beyond the 13 cognitive/behavioral domains:
- `pronse` — behavioral observations section (`_01-00_nse.qmd`)
- `prosirf` — integrated summary (`_03-00_sirf.qmd`)
- `prorecs` — recommendations (`_03-01_recs.qmd`)
- `progen` — general/summary

This means the interpretation stage orchestrates not just domain narratives but also the integrated clinical summary and recommendations — effectively spanning roles `summarize_domain`, `integrate_evaluation`, and `generate_recommendations` in a single batch.

This mapping is especially important in neuropsychological work because the product's value is not merely producing section text; it is preserving structured integration across domains in the final report. The founder's YC application repeatedly emphasizes that integrated interpretation is the distinctive human skill in neuropsychology and the main deficiency of more generic report generators.

---

## Parallel Review Agents

Not all agents in a multi-agent pipeline produce output artifacts — some serve a quality-gate role and run in parallel with production agents without blocking them. The `clinical-validity-reviewer` is a canonical example of this pattern.

The reviewer dispatches **in parallel** with the narrative-writer (or any time before delivery) and operates in a strictly **read-only** mode. It ingests the same CSV and draft QMD files the writer produces, then returns a structured punch list — never modifying files. This design means the main generation thread continues uninterrupted while review happens concurrently.

The reviewer evaluates six axes:
1. **Completeness** — every CSV subdomain has a corresponding narrative file; multi-rater domains cover all raters.
2. **Validity language** — effort/validity test findings trigger required hedging; prohibits definitive language like "malingering" absent positive findings.
3. **Premorbid context** — narrative integrates education, occupation, and baseline; flags missing cultural or sensorimotor caveats.
4. **Score–narrative consistency** — qualitative claims are backed by CSV rows; score ranges match the CSV `range` column exactly.
5. **PHI leak detection** — grep-based scan for names, DOB patterns, SSNs, MRNs (≥6 digits), and phone numbers.
6. **Tone & style** — checks against a supplied style profile; flags bare-assertion clinical claims and unexplained jargon.

The reviewer produces a verdict (`ready_to_sign_out | revise_before_signout | block_signout`) with `CRITICAL_ISSUES`, `MAJOR_ISSUES`, and `MINOR_ISSUES` tiers — mapping directly to orchestrator response logic. If the CSV or QMD directory is unreadable, it immediately returns `block_signout` and halts.

This pattern generalizes: **read-only parallel reviewers** can be attached to any subgraph that produces artifacts, providing quality assurance without introducing serial latency. See [[concepts/report-review-qa]] and [[summaries/clinical-validity-reviewer]] for the full specification.

The `block_signout` verdict is a form of human-in-the-loop gate: a critical issue (PHI leak, fabricated score, validity language error) surfaces to the orchestrator and prevents delivery until resolved.

The YC application makes this review layer even more important. Because the founder positions local handling of sensitive evaluations and preservation of clinical voice as core differentiators, review agents are not optional polish; they are part of the product's trust architecture.

---

## Subagent Specs vs. Dispatchable Agents

A recurring design tension in the Luria system is the distinction between **documentation-as-spec** and **dispatchable subagents**. The four core subagents —

- `Neuropsych_Data_Extractor`
- `PDF_Ingestion_Parser`
- `Narrative_Report_Generator`
- `Sheets_Data_Indexer`

— currently exist as `AGENTS.md + tools.json` specification files under `subagents/`, not as directly invocable agents. To enable parallel Task-tool invocation, these specs would need to be mirrored into `.claude/agents/<name>.md` with appropriate frontmatter.

The cingulate design solves this cleanly by placing subagent definitions directly under `.claude/agents/` from the start, avoiding the spec-vs-dispatchable split. A known hazard flagged in that design: two agent files sharing the same `name:` frontmatter field (`agent-of-cingulate` appears in both `.claude/agents/agent-of-cingulate.md` and `.claude/agents/neuropsych-data-analyst.md`). Subagent name collisions silently break dispatch and must be resolved before running.

Note that `Sheets_Data_Indexer` reflects a now-retired integration: Google Sheets output was removed roughly a month before this writing. The [[concepts/cingulate-engine]] replaced that workflow entirely, using CSV, Arrow, and Parquet as the canonical data formats.

See [[summaries/responses_to_claude]] for the user's direct response to the promotion recommendation.

---

## Shared State Management

A critical implementation detail in any multi-agent pipeline is how agents share and accumulate state. The Luria Starter Kit codifies two key patterns:

- **`Annotated[list, operator.add]` in state** — messages and results accumulate across nodes using a reducer, preventing later nodes from overwriting earlier outputs. This is the correct pattern for fan-in collection of subagent results.
- **Frozen dataclass + `lru_cache` for config** — settings are loaded once and shared as a singleton across all agents, avoiding repeated environment lookups and making the system testable.

These patterns represent the minimal correct approach to state in a LangGraph pipeline. In the 2025 refactor, persistent state is also managed via a SQLite store, and session-level soul context is tracked separately.

The cingulate design externalizes state entirely to `state.json` in the workspace, making it readable by any stage without in-process coupling — a simpler and more durable approach for long-running workflows that may span multiple sessions. This also means an interrupted run can be resumed by re-dispatching the orchestrator: it reads `state.json` and picks up from the first non-`done` stage. See [[concepts/agent-pipeline-state-management]] for broader patterns.

The `ConfigManagerR6` and `ErrorHandlerR6` classes in the `cingulate` package formalize the config-as-singleton and safe-execution patterns at the R level: `ConfigManagerR6$new("config.yml")` loads YAML once, and `ErrorHandlerR6$safe_execute(expr, context)` wraps any expression with logging — the R analog of the frozen dataclass + try/except pattern.

The product narrative in the YC application also suggests a broader principle: orchestration state is part of the clinical record of work, not just runtime bookkeeping. When a system claims near-autonomous workflow execution, explicit state becomes necessary for auditability, resumability, and human oversight.

---

## Subagent Prompt Decoupling

A practical pattern from the starter kit: each subagent's system prompt lives in a standalone Markdown file loaded at runtime rather than embedded in code. This decoupling means:
- Prompts can be edited without touching application code
- Each agent's behavior is documented in a human-readable file
- Prompt files can be version-controlled and diffed independently

The `cingulate` package extends this to prompt *templates*: `inst/prompts/pro*.qmd` files are the authoritative source for domain prompts, and `DomainPrompterR6` assembles messages from them at runtime. Changing a domain's prompt means editing the `.qmd` file only — no R code changes required.

The cingulate subagent design takes this further: each `cingulate-*` subagent is a self-contained `.claude/agents/*.md` file with frontmatter, receiving only its relevant context slice at dispatch time. See [[concepts/subagent-architecture]] for broader patterns in subagent design.

This decoupling is particularly important in clinical settings where style, tone, and interpretive boundaries matter. The founder's YC application highlights "clinical voice" as a non-negotiable requirement, which makes prompt and template separation an architectural necessity rather than just a convenience.

---

## Separate Compiled Graphs

The starter kit separates the IngestGraph and RAGGraph into independently compiled graphs. This pattern scales to larger systems: each pipeline can be tested in isolation, failure is contained, and graphs can be composed by a top-level router node that classifies the incoming request and dispatches to the appropriate subgraph. Adding a router is explicitly called out as the next step when extending the starter:

```
classifier (router) → IngestGraph
                    → RAGGraph
                    → [future subgraphs]
```

As the Luria product vision expands from point tools to an end-to-end workflow system, this separation supports a modular product surface: ingestion, scoring, narrative generation, retrieval, and QA can evolve independently while still participating in a larger orchestrated run.

---

## Data Format Alignment

The [[concepts/cingulate-engine]] defines the canonical data contract for agents in this pipeline. All inter-agent data exchange uses **CSV, Arrow, or Parquet** — not spreadsheet formats. This has two orchestration implications:

1. Any subagent spec referencing Google Sheets output (e.g., `Sheets_Data_Indexer`) is stale and should not be promoted to a dispatchable agent without revision.
2. Parquet is preferred for tabular neuropsychological score data passed between extraction and report-generation agents, given its columnar efficiency and schema enforcement. See [[concepts/parquet-as-knowledge-store]] for details.

The `load_data_duckdb()` function is the canonical entry point for this contract: it reads raw CSVs, computes z-scores, splits by `test_type`, and writes `neurocog.parquet`, `neurobehav.parquet`, and `validity.parquet`. All downstream agents — whether R6 processors, LangGraph nodes, or Claude subagents — consume these Parquet files as their authoritative data source.

Report generation is handled by [[concepts/quarto]] and [[concepts/typst-typesetting]] rather than spreadsheet exports.

This data-contract discipline is one reason orchestration can scale from the founder's original ad hoc R functions to a more autonomous system. The YC application describes a progression from CSV extraction to PDFs to arbitrary file types; orchestration becomes sustainable only when those heterogeneous inputs are normalized into stable machine-readable artifacts.

---

## Structural Stability and Reorganization Risk

A real hazard in multi-agent systems with many moving parts is **structural drift during reorganization** — when folder renames or refactors cause old code paths to resurface or stale specs to re-enter the active codebase. The Luria project experienced this directly when a recent restructuring caused old code to reappear and introduced inconsistencies.

Mitigation strategies:
- Keep agent specs and dispatchable agent definitions co-located or explicitly linked so stale specs are visible.
- Use a canonical directory reference in all documentation and never rely on implicit paths.
- After any structural reorganization, audit `subagents/` and agent definitions in parallel to confirm alignment.
- Update codemaps immediately after refactors to prevent documentation drift.
- Flag subagent name collisions (duplicate `name:` frontmatter) before they cause silent dispatch failures — as documented in the cingulate open questions.
- Gitignore `output/` before any real run to prevent PHI from being committed.
- The `LLM_AGENT_MAP` document (see [[summaries/LLM_AGENT_MAP]]) serves as a stable canonical reference: 12 key files an LLM agent should always know, with their purposes. Maintaining this map is itself a structural stability practice.

As the system is framed externally as an autonomous clinical workflow product, reorganization risk becomes more than a developer inconvenience. Structural drift can undermine the reliability needed for real clinical deployment.

---

## Model Routing in Multi-Agent Systems

Not all agents need the same model. Effective orchestration assigns models by role:

| Role | Model Tier | Rationale |
|---|---|---|
| Intake / validation / deterministic scripts | Small, fast (8B) | Speed and cost; temp=0 for reproducibility |
| Domain narrative prose | Mid-tier (20B) | Clinical fluency without over-engineering |
| Cross-domain reasoning / diagnosis | Large MoE (35B) | Complex inference over full patient profile |
| Patient-facing final output | Largest / best quality | Stakes are highest |
| STT / embeddings | Specialized models | Task-specific architecture |

The cingulate team operationalizes this by assigning opus to the orchestrator, interpretation, and quality-review stages (highest reasoning demands) and sonnet to intake, scoring, and report-writing (more deterministic, structured tasks). See [[concepts/local-llm-inference]] and [[concepts/mixture-of-experts]] for the hardware and model-architecture context behind these choices.

The `OllamaModelRouterR6` supports both Ollama and MLX endpoints, configured via environment variables (`OLLAMA_URL`, `MLX_URL`). The `allow_pull` flag controls whether missing models are auto-pulled — set to `FALSE` in production to prevent unexpected model downloads during a live run. See [[concepts/role-based-llm-routing]] and [[concepts/llm-provider-abstraction]] for the broader patterns.

The starter kit abstracts the LLM behind a `_llm()` helper, making it straightforward to swap between a local endpoint, OpenAI, or Anthropic without restructuring the graph. The cingulate design enforces `mode = "development"` as the default for all LLM calls, switching to `production` only on explicit instruction — a safeguard against accidental live PHI processing.

The YC application makes clear why this matters commercially as well as technically: the founder's position is that generic cloud AI systems are poorly matched to neuropsychological evaluation because they do not respect strict PHI boundaries or the nuanced synthesis required for clinical reporting. Model routing inside a local-first architecture is therefore a core differentiator, not just an implementation detail.

---

## Failure Handling

Robust orchestration requires explicit failure states at the subagent level, not just top-level error handling. The Luria pipeline defines four states per domain agent:

| State | Meaning | Orchestrator Response |
|---|---|---|
| `COMPLETED` | All artifacts produced | Continue |
| `DEGRADED` | Some artifacts missing | Include available, flag gap |
| `SKIPPED` | No data for this domain | Insert "Not Administered" placeholder |
| `FAILED` (retryable) | Model timeout, transient error | Retry up to 2× with backoff |
| `FAILED` (non-retryable) | Corrupt data | Skip domain, add to issue list |

The cingulate runbook maps stage outcomes to four statuses: `DONE | DONE_WITH_CONCERNS | NEEDS_CONTEXT | BLOCKED`. The orchestrator halts on `BLOCKED` and surfaces the failing stage, a one-line reason, and the absolute path to the stage log. Common failure modes and their fixes are documented as a lookup table:

| Symptom | Likely Cause | Fix |
|---------|--------------|-----|
| `BLOCKED` at scoring: "JEP 66 handshake failed" | R startup timeout | `.rs.restartR()`, avoid auto-loading `library(cingulate)` |
| `BLOCKED` at interpretation: "model not available" | Ollama not running | `bash setup_ollama.sh` |
| `BLOCKED` at report: "extension not found" | Missing Quarto extension | `quarto add` or symlink extensions |
| `BLOCKED` at qa: "pdftotext not found" | Poppler missing | `brew install poppler` |
| `DONE_WITH_CONCERNS` at interpretation | LLM empty/garbled output | Re-dispatch with failed domain list, switch `llm_mode` |

The cingulate orchestrator takes a stricter position than the Luria pipeline: **no auto-retry at all**. Any `error` state halts the chain and surfaces to the human reviewer, who manually re-dispatches after investigating. This is appropriate for a scaffolding-phase system where failure modes are not yet fully characterized, and it reflects the general principle that in clinical contexts, silent retries could mask data errors.

Any validity flag from a domain pauses the pipeline and surfaces the issue to a human — critical in clinical contexts where a missed result has real consequences.

The YC application indirectly strengthens this design choice: when the product promise is nearly autonomous execution of a sensitive clinical workflow, visible failure states are preferable to hidden retries because they preserve clinician control and accountability.

---

## Human-in-the-Loop Checkpoints

Long-running multi-agent pipelines benefit from defined human review gates:
- After PII redaction (confirm no PHI leaks before downstream agents see data)
- After quality review before final delivery
- When any domain returns a validity flag
- When the parallel reviewer returns `block_signout` (PHI leak, fabricated score, or validity language error detected)
- After a maximum retry threshold is exceeded (e.g., 3 correction cycles → human escalation)
- **Before any subagent is invoked against a real case** — the cingulate design makes this an explicit approval gate: the clinician reviews before any live PHI runs

Critically, the cingulate team **never approves a report** — that is always a human decision. QA issue severities (`blocker | major | minor | nit`) map to required actions: blockers must be resolved before any sign-off. This is especially important in [[concepts/phi-data-handling]] contexts where automated errors carry regulatory and clinical risk.

This hybrid posture also matches the founder narrative in the YC application. Luria is framed as automating the workflow, but not eliminating the importance of expert integration and clinical judgment. In practice, orchestration succeeds in healthcare when it amplifies expert throughput rather than pretending to remove the expert entirely.

---

## Open Questions Across the System

Several unresolved items span the cingulate scaffolding and broader Luria architecture:

- **CWD constraint:** Several cingulate helpers require `setwd(workspace_path)` because they don't accept a workspace-path argument. A clean refactor is flagged but out of scope for current scaffolding.
- **Subagent name collision:** Two agent files both declare `name: agent-of-cingulate` in frontmatter. One must be renamed to `neuropsych-data-analyst` to match its filename.
- **LLM mode policy:** When to flip from `development` to `production` for real cases is a human policy decision.
- **PHI handling:** `output/` must be confirmed gitignored before any real run.
- **Test fixture:** A synthetic case under `output/_fixture/` is recommended for smoke-testing the chain without real PHI. See [[concepts/smoke-test-scripts]] for related patterns.
- **`allow_pull` policy:** The router's `allow_pull = FALSE` default is safe for production but requires manual model management; a model availability check via `check_available_models()` should be part of any pre-run checklist.
- **Commercial packaging question:** The YC application leaves company, pricing, traction, and deployment packaging largely unspecified, so the operational boundary between internal workflow engine and productized customer-facing system remains open.

---

## Priority Order for Building Multi-Agent Pipelines

From the Luria plan, a practical build sequence:
1. **Start with a minimal sequential pipeline** — validate end-to-end before adding concurrency.
2. **Prove the per-domain pattern on one domain first** — validate the full DataPrep → Text/Table/Figure → render chain before fanning out.
3. **Wire the fan-out node** — extend the graph with the parallel spawning mechanism.
4. **Add the join/collect node** — aggregate domain results with explicit state management.
5. **Layer in failure handling** — add `DEGRADED`/`SKIPPED`/`FAILED` states before going to production.
6. **Attach read-only parallel reviewers** — quality gates like the clinical-validity-reviewer can be wired in at this stage without disrupting the main generation thread.
7. **Add voice/style subagents last** — aesthetic enhancements are not blockers; ship the functional pipeline first.
8. **Create a synthetic fixture case** — smoke-test the full chain against a fake patient before touching real PHI. See [[concepts/smoke-test-scripts]] for related patterns.

The YC application provides a useful origin story for why this order works. The system began as small practical utilities for one clinician's workflow, then accumulated automation, templating, and model assistance over several years. That organic path mirrors the recommended build order: start narrow, prove utility, then orchestrate at larger scale.

---

## Related Concepts

- [[concepts/langgraph-agent-workflows]] — the specific orchestration runtime (LangGraph StateGraph) used in the Luria pipeline
- [[concepts/subagent-architecture]] — design patterns for individual subagents within the orchestrated system
- [[concepts/cingulate-engine]] — the core data processing engine defining canonical data formats (CSV/Arrow/Parquet) for inter-agent exchange
- [[concepts/local-llm-inference]] — the local model serving layer used across agents
- [[concepts/pii-redaction-pipelines]] — the redaction gate that all downstream agents depend on
- [[concepts/retrieval-augmented-generation]] — used in RAGGraph queries and domain lookup
- [[concepts/neuropsychological-reporting]] — the clinical domain driving this orchestration architecture
- [[concepts/persistent-memory]] — cross-session patient context management
- [[concepts/mixture-of-experts]] — the MoE architecture behind large models used for high-stakes reasoning
- [[concepts/phi-data-handling]] — HIPAA-aware permission model restricting which agents can access raw patient data
- [[concepts/parquet-as-knowledge-store]] — Parquet as the preferred format for tabular data passed between pipeline agents
- [[concepts/quarto]] — report generation layer consuming agent outputs
- [[concepts/typst-typesetting]] — typesetting backend for final clinical report rendering
- [[concepts/report-review-qa]] — quality assurance patterns for neuropsych report review
- [[concepts/validity-language]] — standards for hedging effort and validity findings in clinical narratives
- [[concepts/redaction-tokens]] — placeholder tokens used to mask PHI in agent-visible content
- [[concepts/per-patient-workspace]] — the workspace-as-contract pattern used in the cingulate agent team design
- [[concepts/edit-protection-pattern]] — the `# manual-edit` marker convention protecting hand-edited QMD files
- [[concepts/skills-modules]] — the `luria-*` skill layer that cingulate subagents wrap
- [[concepts/luria-neuropsych-pipeline]] — the full clinical pipeline that these orchestration patterns serve
- [[concepts/agent-pipeline-state-management]] — patterns for externalizing and managing stage state across sessions
- [[concepts/smoke-test-scripts]] — pre-run verification and synthetic fixture case patterns
- [[concepts/luria-skills]] — the named skill definitions delegated to by the orchestrator
- [[concepts/role-based-llm-routing]] — the YAML-configured role-to-model mapping used by OllamaModelRouterR6
- [[concepts/llm-provider-abstraction]] — patterns for swapping LLM backends without restructuring pipeline logic
- [[concepts/fallback-strategy]] — graceful degradation when specialized models are unavailable
- [[concepts/artifact-caching-pipeline]] — content-addressed caching of stage outputs in the bridge layer
- [[concepts/domain-processor-pattern]] — the R6 factory pattern for domain-specific data processing
- [[concepts/r6-class-architecture]] — the R6 class system underlying OllamaModelRouterR6, ReportLLMBridgeR6, and related components
- [[concepts/yaml-configuration]] — role-to-model mapping and workspace configuration via YAML files
- [[concepts/neuropsychological-assessment-automation]] — the broader automation target that orchestration makes operational
- [[concepts/neuropsychological-assessment-pipeline]] — the end-to-end pipeline structure being coordinated
- [[concepts/neuropsychological-assessment-workflow]] — the clinical workflow decomposition that motivates the orchestration design
- [[concepts/clinical-ai-copilot]] — a useful contrast for systems that assist a clinician without fully orchestrating the workflow
- [[concepts/clinical-ai-reasoning]] — the higher-order interpretive layer delegated to specialized reasoning agents
- [[concepts/clinical-narrative-generation]] — narrative drafting as one stage within a larger orchestrated system
- [[concepts/local-first-architecture]] — local execution and storage constraints that shape orchestration boundaries
- [[concepts/privacy-first-software]] — privacy as a first-class product and architecture requirement
- [[concepts/clinical-data-privacy]] — operational privacy requirements affecting agent access patterns
- [[concepts/clinical-data-management]] — structured handling of source records and derived artifacts across the pipeline
- [[concepts/narrative-report-generation]] — report-writing stage design in multi-agent systems
- [[concepts/modular-report-architecture]] — report section decomposition that aligns with per-domain subagents

See also: [[summaries/AGENTS_luria]] · [[summaries/SKILL]] · [[summaries/responses_to_claude]] · [[summaries/clinical-validity-reviewer]] · [[summaries/SESSION_SUMMARY_2025-04-28]] · [[summaries/2026-04-26-cingulate-agent-team-design]] · [[summaries/agent-team]] · [[summaries/neuropsych-narrative-writer]] · [[summaries/neuropsych-pdf-parser]] · [[summaries/README]] · [[summaries/DEPENDENCIES]] · [[summaries/LLM_AGENT_MAP]] · [[summaries/CLAUDE]] · [[summaries/Apply-to-Y-Combinator-JWT]]

See also: [[summaries/Introducing-FrontierCode]]

See also: [[summaries/redesign_20260623110817]]

See also: [[summaries/redesign_20260623110910]]

See also: [[summaries/Luria_AI_Q4_Investor_Memo_2026]]