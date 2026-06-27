---
sources: [summaries/README_20260413235533.md, summaries/README_20260413235353.md, summaries/Luria_AI_Q4_Investor_Memo_2026.md, summaries/LLM Benchmark Comparison.md, summaries/File Folder Structure Rebuild.md, summaries/Apply-to-Y-Combinator-JWT.md, summaries/redesign_20260623110910.md, summaries/redesign_20260623110817.md, summaries/README.md, summaries/installation.md]
brief: Luria is a local-first neuropsych AI platform for end-to-end clinical workflows.
---

# Luria Overview

Luria is a local-first platform for neuropsychological assessment workflows, clinical data management, and AI-assisted report generation. It is designed to support the full evaluation process from intake through synthesis and final reporting, with particular emphasis on privacy-sensitive clinical work such as pediatric and forensic neuropsychology. Across the project materials and investor narrative, Luria is framed not merely as a report-writing tool but as an end-to-end neuropsychological AI platform intended to automate scoring, interpretation, documentation, and clinician workflow while preserving diagnostic nuance and strict PHI boundaries.

The platform combines structured data pipelines, AI-assisted reasoning, local and optional cloud language model support, optional R-based statistical and visualization workflows, and an evolving application architecture that is being reorganized into clearer service boundaries. In the YC application summary ([[summaries/Apply-to-Y-Combinator-JWT]]), Joey Trampush describes it as a local-first, agent-based system that can operate with teams of agents and subagents to carry out clinical workflow steps nearly autonomously while preserving strict PHI boundaries. The Q4 2026 investor memo ([[summaries/Luria_AI_Q4_Investor_Memo_2026]]) sharpens that framing into a market and product thesis: Luria aims to replace 8–12 hours of manual neuropsychological scoring, interpretation, and report writing with a production-ready workflow that can generate clinician-grade outputs in under an hour while maintaining high accuracy.

That framing complements the broader technical architecture documented elsewhere in the project and helps explain why Luria emphasizes local inference, synthesis across domains, clinician-grade narrative output, ongoing [[concepts/codebase-reorganization]], and a more modular platform architecture. It also places Luria within the broader domains of [[concepts/neuropsychological-assessment-automation]], [[concepts/clinical-ai-copilot]], [[concepts/clinical-ai-reasoning]], and [[concepts/healthcare-workforce-automation]].

## Core Purpose

Luria targets solo clinicians, neuropsychologists, healthcare systems, and researchers who need to:
- Ingest and parse neuropsychological PDF reports with local PHI redaction
- Extract and manage neuropsychological test scores from raw data sources
- Generate structured clinical narrative reports
- Leverage LLM assistance while maintaining privacy and local-first data handling
- Integrate Python and R-based statistical and visualization workflows
- Query a private knowledge base via retrieval-augmented generation
- Engage an AI clinical copilot for case reasoning grounded in source citations
- Support integrated interpretation across neurocognitive and neurobehavioral domains
- Automate the workflow from case intake through scoring, interpretation, quality review, and final reporting

In practical terms, Luria is oriented toward the parts of neuropsychological work that are most difficult to scale manually: assembling records, extracting scores, organizing background history, checking score validity, synthesizing patterns across domains, supporting differential diagnosis, and drafting reports in an authentic clinical register. This makes it closely related to [[concepts/neuropsychological-assessment-workflow]], [[concepts/neuropsychological-assessment-automation]], [[concepts/clinical-ai-copilot]], [[concepts/narrative-report-generation]], and [[concepts/neuropsychological-score-interpretation]].

The investor memo also makes explicit that Luria is meant to address a workforce bottleneck, not just a documentation inconvenience: neuropsychologists are scarce, report writing is time-intensive, and documentation quality can vary by clinician experience. Luria's core purpose is therefore both operational and clinical—expanding specialist capacity while keeping output quality high.

---

## Founder and Product Context

Luria emerges from direct clinical need rather than from a generic software-first concept. In [[summaries/Apply-to-Y-Combinator-JWT]], Joey Trampush describes building the system out of his own work as a researcher in neurodevelopment and psychiatric genetics, a pediatric neuropsychologist, and founder of a Los Angeles-based pediatric and forensic neuropsychology practice. He reports that increased post-pandemic patient volume made conventional evaluation and report-writing workflows increasingly unsustainable, especially because report writing is both labor-intensive and the central clinical deliverable.

The Q4 2026 investor memo ([[summaries/Luria_AI_Q4_Investor_Memo_2026]]) extends this founder narrative into a clearer commercialization story. There, Luria is presented as having moved from prototype to production-ready validation, with 47 active cases processed in Q4 2026, clinician-validated report quality of 94%, and an 89% reduction in per-case time from 8.5 hours to 55 minutes. The memo claims this performance exceeds manual junior-clinician documentation accuracy while preserving diagnostic specificity, reinforcing that the product is meant to augment or compress specialist labor rather than simply produce generic drafts.

The documents together describe the product's development path:
- early use of R and RMarkdown for clinical data processing and reporting
- expansion into functions for extracting data from CSVs, PDFs, and eventually other file types
- later use of LaTeX, Quarto, and Typst for report production
- eventual incorporation of GitHub Copilot and LLMs into the workflow
- progression toward a multi-agent system able to perform much of the workflow autonomously
- transition from a prototype toolchain into a platform intended for healthcare-system pilots and enterprise deployment

This background helps explain several recurring design choices in Luria:
- strong preference for [[concepts/local-first-architecture]] and [[concepts/privacy-first-software]]
- focus on [[concepts/clinical-communication-register]] and clinician-quality narrative output
- emphasis on integrated interpretation rather than isolated test summaries
- support for pediatric and forensic workflows, including [[concepts/forensic-neuropsychological-evaluation]]
- retention of optional R-based tooling through [[concepts/r-neuropsych-packages]], [[concepts/quarto]], and [[concepts/typst-typesetting]]
- a cautious, staged approach to architectural change consistent with [[concepts/migration-strategy]]
- growing attention to enterprise readiness, compliance, validation, and scale

---

## Luria.app — The Redesigned Clinical Interface

The 2026 redesign (documented across [[summaries/redesign_20260623110817]] and [[summaries/redesign_20260623110910]]) introduces a purpose-built desktop and mobile application with a structured sidebar workspace and AI reasoning throughout every phase of the assessment workflow. The redesign is illustrated through a complete demo case — "Biggie Smalls," a 7-year-old male evaluated for ADHD and dysgraphia — flowing through every module.

The redesigned interface also aligns with both the YC framing and the investor memo's product claim that Luria can support a full workflow from "soup to nuts," not just document generation. Rather than treating intake, document review, synthesis, and drafting as separate utilities, the UI makes them contiguous phases of a single case-centered workflow. This is consistent with the investor memo's emphasis on end-to-end orchestration from intake to scoring to interpretation to reporting.

### Sidebar Navigation

Luria.app is organized into a consistent sidebar with three sections (keyboard shortcuts `⌘1`–`⌘7`):

- **Home:** Report Workspace
- **Workspace:** Patient Intake, Clinical Office, Console & Synthesis
- **Tools:** Visuomotor Sketchpad, Cognitive Map, Report Builder

Each session is anchored to an **Active Case-File** showing patient demographics, test count, and active model (e.g., oMLX Primary, M3 Max). A global `⌘K` launcher provides quick access throughout.

### Application Pages

#### Report Workspace (`⌘1`)
The dashboard hub displaying all active dossiers, a patient header, a 5-step **Progress Tracker** (Intake → Documents → Cognitive Map → Synthesis → Report), and live neurocognitive/neurobehavioral score tables. A **Synthesis Panel** surfaces AI-generated pattern impressions in real time. Supports Light, Dark, and Amber themes. A mobile companion view mirrors the current phase and domain color coding.

Example score table from the demo case (Biggie Smalls, 7y, ADHD & dysgraphia):

| Domain | Score | Status |
|---|---|---|
| General Cognitive | 96 | Average |
| Academic Skills | 78 | Clinical Concern |
| Memory | 88 | Borderline |
| Attention/Executive | 76 | Clinical Concern |
| ADHD/Exec (T-score) | T72 | Elevated |

The investor memo adds an operational interpretation of this workspace: the point is not only visibility but throughput. Luria is intended to support case volume growth without degrading documentation quality, making dashboarded workflow state and rapid report assembly central product features rather than UI conveniences.

#### Patient Intake (`⌘2`)
Structured history capture with auto-tagging by the **Λ (Lambda) AI assistant**. Referral text is parsed into discrete clinical concerns with tags (e.g., `Inattention`, `Graphomotor / handwriting`, `r/o ADHD`, `r/o Dysgraphia`). Developmental and medical history is organized into a flagged table. The AI assistant identifies workflow gaps (e.g., missing prior psychoeducational testing) and suggests follow-up items. See [[concepts/neurodevelopmental-clinical-intake]] and [[concepts/staged-clinical-intake]] for related patterns.

This page also reflects the platform's larger goal of reducing administrative burden before formal interpretation begins. In Luria's broader product logic, workflow automation starts at intake, not just at final drafting.

#### Document Ingestion (`⌘3`, Page 03)
PDF and file ingestion using markitdown (PDF → Markdown conversion). Provides a visual upload interface for clinical records. See [[concepts/pdf-data-extraction]] and [[concepts/ocr-pipeline]].

#### Clinical Office / Documents (`⌘3`, Page 04)
Indexed source documents with RAG-powered fact extraction (24 chunks embedded in the demo). Each extracted fact is linked to its source page. Status indicators: `INDEXED`, `PROCESSING`, `NOT ON FILE`. Facts pre-fill the report background section. Source types include Clinical Interview, Prior Medical Records, School Records/IEP, and Prior Testing. Related: [[concepts/retrieval-augmented-generation]], [[concepts/clinical-data-management]].

#### Console & Synthesis (`⌘4`)
The core AI reasoning interface — described as "a clinical copilot you argue with." Clinicians pose diagnostic questions; **Λ Luria** responds with grounded multi-source reasoning and numbered citations mapped to an Evidence Rail on the right. Example exchange:
- *Question:* Is inattention primary ADHD or secondary to dysgraphia/academic frustration?
- *Luria's answer:* Primary ADHD — citing cross-setting Conners-4 elevations (parent T=72, teacher T=70), dissociation from output scores (NEPSY-II SS 76 independently depressed), and onset history predating writing demands per the caregiver interview.

Actions: `[Add to Synthesis]`, `[Draft this section]`, `[Show reasoning]`. The evidence rail lists each cited source with scores and status (`pending` for tests not yet administered). See [[concepts/clinical-ai-reasoning]] and [[concepts/neuropsychological-synthesis]].

This page reflects one of Luria's clearest differentiators: not merely summarizing tests one by one, but helping clinicians integrate evidence across domains in a way consistent with neuropsychological formulation. That concern is emphasized in [[summaries/Apply-to-Y-Combinator-JWT]], where the founder contrasts Luria with publisher-generated reports that remain specific to isolated instruments and lack cross-domain integration. The investor memo reinforces this same point by highlighting domain-specific reasoning chains across eight neuropsychological domains and an interpretation engine with differential diagnosis support.

#### Visuomotor Sketchpad (`⌘5`)
A drawing/annotation tool available in Light and Dark themes, used for graphomotor assessment tasks (e.g., Beery VMI). Related: [[concepts/dysgraphia]].

#### Cognitive Map — Amber Theme (`⌘6`)
A constellation visualization of 13 cognitive domains, sized by test breadth and colored by severity. The Amber theme is a dark, single-purpose interface for profile reading at a glance. AI auto-detects co-varying clusters and names the pattern. From the demo:
> **Frontal-graphomotor cluster** — Attention, executive control, and motor output co-vary and are jointly depressed while verbal reasoning is spared — the ADHD-Inattentive + dysgraphia signature. Convergence: 4/4. Spared strengths: Verbal 102, Reasoning 96.

See [[concepts/cognitive-domains]], [[concepts/neuropsychological-score-interpretation]], [[concepts/adhd-clinical-features]].

#### Report Builder (`⌘7`)
Section-by-section report drafting with AI assistance. 6-section structure (Reason for Referral, Background & History, Tests Administered, Results by Domain, Summary & Formulation, Recommendations). Controls include voice tone (Clinical / Balanced / Parent), reading level slider (Grade 8 ↔ Professional), and insertable elements (score table, domain figure, DSM-5 criteria). The AI drafts from all available tests and source documents with citations. Related: [[concepts/narrative-report-generation]], [[concepts/clinical-report-structure]], [[concepts/modular-report-architecture]].

The emphasis on tone, register, and integrated narrative also matches the founder's claim that "clinical voice" is non-negotiable in this domain and that generic cloud AI tools are poorly matched to that requirement. In the investor memo, this section's value is translated into measurable outcomes: 94% clinician-validated report quality and markedly lower time per case.

---

## LLM Infrastructure & Local Inference

The redesign documents the full technical agent pipeline in three views (Architecture Overview, Fallback Chain, and Data Flow).

The YC materials further reinforce that this architecture is a strategic product choice, not an implementation accident: local-first execution is presented as essential for handling sensitive evaluations, and agent teams are described as the mechanism for automating the workflow nearly end to end. The investor memo adds that Luria now uses a multi-model architecture combining local and cloud LLMs for quality/cost optimization, while keeping privacy controls central and preserving a HIPAA-aligned deployment posture.

### Architecture Overview

- **UI components:** `IntakeDossier.tsx`, `ReportJobStatus`, `ConsoleChat`
- **Orchestration:** `redactPhi()` PHI guard on `/stt/summarize`, `reportJobs.ts` (`createReportJob()` / `runReportJob()`), `llmAbortContext` via AsyncLocalStorage for mid-job cancellation
- **Agents:** `agentRunner.ts` → section agents (`nseCodSummary`, `ROCFT`, report-section agents)
- **Client:** `LocalFallbackLLMClient` with `pickProvider()`

See [[concepts/multi-agent-orchestration]], [[concepts/agent-pipeline-state-management]], [[concepts/phi-deidentification-pipeline]].

### Fallback Chain (Priority Order)

1. **oMLX** — local OpenAI-compatible API (PHI-safe) → `queryOpenAICompatible`
2. **vMLX** — local Responses API (PHI-safe) → `queryVMLX`
3. **Ollama** — local native API (PHI-safe) → `queryOllamaNative`
4. **Cloud** — remote fallback, non-PHI only (blocked when `restrictToPreferredProviders: true`)

The local-only gate keeps all patient data on-device; cloud is reachable only for non-PHI requests. The `LLMGenerateRequest` struct carries `messages`, `temperature`, `maxTokens`, and `restrictToPreferredProviders` fields. `llmAbortContext` uses `AsyncLocalStorage` to thread an abort signal through `generate()` so a job can be cancelled mid-fallback.

See [[concepts/local-llm-inference]], [[concepts/llm-provider-abstraction]], [[concepts/fallback-strategy]], [[concepts/omlx-server]], [[concepts/local-first-architecture]].

### Data Flow

**Path A — Intake → Inference:**
`IntakeDossier.tsx` → encrypted SQLite → `redactPhi()` → local LLM → Clinical Summary

**Path B — UI → Report Job:**
`ReportJobStatus` → `POST /api/pipeline/orchestrate-report` → `createReportJob()` → `runReportJob()` → section agents → fallback chain

**Path C — Agent Context → Result (`agentRunner.ts#35-44`):**
`AgentContext` (raw clinical data + patient scores) → section agent → `generate()` → structured report section

This architecture supports Luria's core operating model: patient-facing inference remains local when PHI is present, while more general reference and tooling layers can remain modular and provider-agnostic. The investor memo's description of local-first processing, privacy-preserving architecture, and real-time clinician feedback loops suggests that these infrastructure patterns are now being positioned as production features rather than only technical preferences.

---

## Knowledge Base Integration — OpenKB (Planned)

The redesign proposes integrating OpenKB as the backend knowledge layer (not yet implemented in the app). OpenKB is an open-source CLI system that compiles raw documents into a structured, interlinked wiki-style knowledge base using LLMs, powered by PageIndex for vectorless long-document retrieval.

**Why not traditional RAG?** Traditional [[concepts/retrieval-augmented-generation]] rediscovers knowledge from scratch on every query; nothing accumulates. OpenKB compiles knowledge once into a persistent wiki and keeps it current. Cross-references already exist; contradictions are flagged; synthesis reflects everything consumed. See [[concepts/knowledge-base-architecture]], [[concepts/knowledge-continuity]].

**Document handling:**

| | Short documents | Long documents (PDF ≥ 20 pages) |
|---|---|---|
| Convert | markitdown → Markdown | PageIndex → tree index + summaries |
| LLM reads | Full text | Document trees |
| Result | summary + concepts | summary + concepts |

**Knowledge compilation steps:** When a document is added, the LLM generates a summary page, reads existing concept and entity pages, creates or updates concepts with cross-document synthesis, creates or updates entity pages, and updates the index and log. A single source may touch 10–15 wiki pages.

**Generators:**
- `openkb query` — grounded Q&A with citations (persist with `--save`)
- `openkb chat` — multi-turn session with slash commands (`/add`, `/skill new`, `/save`, `/lint`, `/status`, `/clear`)
- `openkb skill new` — compiles wiki subset into a portable Anthropic Skill installable by Claude Code, Codex CLI, Gemini CLI, and Cursor
- `openkb skill eval` / `validate` / `history` / `rollback` — quality gates and versioning

**Configuration** (`openkb init` → `.openkb/config.yaml`):
```yaml
model: gpt-5.4
language: en
pageindex_threshold: 20
```
Model names use LiteLLM `provider/model` format. `entity_types` is an optional YAML list overriding the default entity-type vocabulary.

The OpenKB stack: markitdown, OpenAI Agents SDK, LiteLLM, Click, and watchdog.

See [[concepts/knowledge-capture]], [[concepts/luria-skills]], [[concepts/skills-modules]].

In the broader Luria vision, this knowledge layer complements rather than replaces case-specific reasoning: OpenKB accumulates reusable knowledge, while Luria applies that knowledge within local, patient-specific workflows. The investor memo's mention of a knowledge base for continuous learning fits this same architecture, suggesting that reusable institutional knowledge and per-case automation are intended to reinforce one another.

---

## Python Application Architecture (Original Layer)

### Streamlit Application

The original Luria Python application remains a Streamlit desktop UI (served at `http://127.0.0.1:8501`) with four functional tabs:

1. **Ingest** — Drag-and-drop neuropsych PDFs through a 4-stage pipeline: parse → extract → index → report
2. **Ask** — Chat interface for RAG Q&A using only ingested data; combines SQL filtering with semantic search
3. **Knowledge Base** — Browse indexed clinical summaries and test-score tables with filters by `doc_id` and cognitive domain
4. **Audio** — Upload audio files (m4a/mp3/wav), transcribe via MacWhisper CLI, summarize via local oMLX server

See [[summaries/README_luria]] for the full application README.

### Luria KB (Knowledge Layer)

Luria KB is a dedicated knowledge repository that agents query before reasoning or writing. Its purpose is to supply clinical reference material — diagnostic criteria, scoring guidance, [[concepts/validity-language]], report-writing standards — in a retrieval-ready format.

KB is organized as a **two-tier** system:

**Tier 1 — `wiki/` (Curated Markdown KB)**
- ~40 hand-authored Obsidian-style markdown notes, cross-linked with wikilinks
- 9 topic pillars + 31 concept notes covering DSM-5-TR diagnostic criteria, score classifications, [[concepts/validity-language]], recommendations writing, HIPAA/APA compliance, ICD-10 coding, base rates, RCI
- Queried on **every** report via long-context with prompt caching (not embeddings)

**Tier 2 — `store/` (Specialized Sub-Corpora)**
- `store/typst/` — Vendored Typst docs for report templates
- `store/quarto/` — Vendored Quarto docs for R/Quarto pipelines
- `store/shared_references/` — Legacy clinical content scheduled for restructuring

### Ingest Pipeline

Orchestrated by [[concepts/langgraph-agent-workflows]] as a `StateGraph` with four nodes:

- **Parse** — [[concepts/docling-pdf-parsing]] extracts text and layout; PHI is redacted locally before any network call
- **Extract** — Claude Sonnet structures the narrative into JSON (test scores, clinical summaries)
- **Index** — SQLite (`data/neuropsych.db`) and [[concepts/lancedb-vector-store]] store structured and vector data locally
- **Report** — Generates a markdown narrative report; optional Typst or Quarto rendering for print-ready PDFs

See [[concepts/agent-pipeline-state-management]], [[concepts/neuropsychological-assessment-pipeline]].

### Agent Roles

1. **Diagnosis agent** → queries wiki tier for diagnostic criteria and classification guidance
2. **Recommendations agent** → queries wiki tier for evidence-based recommendations
3. **Report-writing agent** → queries `typst/` and `quarto/` stores for template and formatting guidance

See [[concepts/multi-agent-orchestration]] and [[summaries/AGENTS_luria]].

This older Python/Streamlit layer shows the product's historical roots: what began as modular tooling for parsing, extraction, indexing, and reporting has been evolving toward a more unified clinical workspace and a more explicit multi-agent orchestration model. The investor memo suggests that this evolution has reached a new phase: rather than primarily serving as an internal tooling stack, Luria is now being articulated as a deployable clinical product for healthcare-system pilots.

---

## Repository Reorganization and Emerging Service Architecture

The "File Folder Structure Rebuild" plan ([[summaries/File Folder Structure Rebuild]]) adds an important architectural layer to understanding Luria's trajectory. It documents a safe, local-only, idempotent shell-script refactor intended to reorganize the repository without deleting unknown files. Rather than changing application behavior directly, the script creates a new target structure and moves or copies known components into clearer buckets.

The proposed target architecture includes:
- `app/streamlit` for the Streamlit UI
- `services/` for ingest, retrieval, agent workflows, storage, voice, reporting, and integrations
- `api/fastapi/routes` for service endpoints
- `data/` for documents, vectors, and databases
- `voice_assets/` for brand, style, and soul resources
- `external/` for preserved legacy or experimental systems
- `tests/` and `scripts/` for support infrastructure

This refactor is significant because it makes explicit a transition already implicit elsewhere in the project: Luria is moving from a more monolithic Python app layout toward a modular service-oriented structure. The document maps key legacy modules into new destinations:
- Streamlit app → `app/streamlit`
- `neuropsych_rag` → `services/retrieval`
- `neuropsych_agent/graph.py` → `services/agent/orchestrator.py`
- `neuropsych_agent/nodes.py` → `services/agent/workflows/ingest_flow.py`
- selected tools → storage, voice, reporting, and integrations service areas
- `voice/` → `voice_assets/`
- `agent/cingulate` → `external/cingulate`
- `rag/*` → `external/experimental`

The document also clarifies what is deliberately left unresolved after the filesystem migration:
- imports are not yet rewritten
- duplicated logic is not removed
- competing pipelines are not reconciled
- `nodes.py` is only staged, not fully modularized

This staged approach is consistent with [[concepts/migration-strategy]], [[concepts/repository-hygiene]], and [[concepts/python-project-structure]]. It suggests that Luria's architecture should be understood in two layers at once:
1. the currently functioning historical Python/Streamlit and agent pipeline implementation
2. the intended future layout, where UI, services, data, voice assets, integrations, and legacy systems are more cleanly separated

The investor memo adds a business rationale for this architectural maturation. Goals such as enterprise deployment infrastructure, SOC 2 Type I certification, healthcare-system pilots, and a multi-site validation study imply that repository cleanup and service separation are not merely engineering hygiene—they are prerequisites for deployment, compliance, and scale.

In other words, the repository rebuild is not a side note; it is evidence that Luria is actively being reshaped into a more maintainable platform architecture capable of supporting the larger clinical and agentic ambitions described throughout the project.

---

## Privacy & Deployment: Local-First Architecture

Luria enforces its privacy boundary at the **data layer**, not by policy:

| Data Class | Where It Runs | Reason |
|---|---|---|
| **Patient data** (PHI) — case histories, score data, draft reports | **Local only** — Ollama/MLX | HIPAA compliance; PHI never leaves device |
| **Reference material** — wiki, diagnostic criteria, scoring tables | Cloud APIs allowed | Public knowledge; no patient information |

In the Luria.app redesign, the `restrictToPreferredProviders: true` flag in `LocalFallbackLLMClient` implements this boundary at the code level — cloud providers are blocked for any request carrying PHI.

The repository rebuild guidance reinforces the same privacy philosophy operationally: architectural changes should be performed locally, with local backups or git snapshots first, and without granting external access. This mirrors the product's broader commitment to [[concepts/local-first-architecture]], [[concepts/privacy-first-software]], and [[concepts/clinical-data-privacy]].

This privacy model is also central to the market thesis articulated in both [[summaries/Apply-to-Y-Combinator-JWT]] and [[summaries/Luria_AI_Q4_Investor_Memo_2026]]. In the latter, the company explicitly cites local LLMs as an enabling technology for privacy-preserving clinical AI and presents HIPAA-aligned architecture as a differentiator. Luria's local-first design is therefore part of both its technical architecture and its competitive positioning.

See [[concepts/local-first-architecture]], [[concepts/phi-data-handling]], [[concepts/pii-redaction-pipelines]], [[concepts/privacy-first-software]], [[concepts/phi-deidentification-pipeline]], [[concepts/clinical-data-privacy]], and [[concepts/healthcare-ai-regulation]].

---

## Competitive Positioning and Clinical Differentiation

Luria is positioned as a specialized neuropsychology product rather than a generic medical scribe, note generator, or publisher-specific scoring utility. The YC application summary describes its intended differentiation in three main ways:

1. **Full-workflow support** rather than narrow report templating
2. **Integrated cross-domain interpretation** rather than isolated test-by-test commentary
3. **Strictly local handling of sensitive patient data** rather than default cloud inference

The founder contrasts Luria with tools from large test publishers, which may produce polished but generic reports tied to specific measures or batteries. Luria instead emphasizes synthesis across cognitive, academic, behavioral, and developmental data sources in a clinician-quality voice. That positioning aligns the product with [[concepts/neuropsychological-reporting]], [[concepts/neuropsychological-synthesis]], [[concepts/clinical-narrative-generation]], and [[concepts/clinical-ai-reasoning]].

The Q4 2026 investor memo adds additional commercial differentiation claims:
- 94% clinician-validated report quality
- higher accuracy than manual junior clinician work (94% vs 87%)
- 89% time savings per case
- no degradation in diagnostic specificity
- active pilot negotiations with healthcare systems

It also frames Luria as a response to specialist scarcity, manual workflow bottlenecks, inconsistent documentation quality, and burnout-driven attrition. This situates Luria not only as a neuropsychology-specific reporting system but as a vertical clinical AI platform addressing a workforce-capacity problem. In that sense, Luria should be understood as a specialized clinical deployment of [[concepts/human-in-the-loop-clinical-ai]] and [[concepts/healthcare-workforce-automation]], adapted to the interpretive demands, privacy requirements, and narrative conventions of neuropsychology.

---

## Key Integrations

### LLM Support
Luria supports both cloud-based APIs and local LLM inference via [[concepts/omlx-server]] (OpenAI-compatible local server). See [[concepts/llm-provider-abstraction]], [[concepts/local-llm-inference]], [[concepts/mlx-framework]], [[concepts/openai-compatible-api]].

### Luria Voice (Optional)
Clinician-specific reporting through three layers:
- **BRAND** — `_brand.yml` for logos, colors, and typography
- **SOUL** — style profile and exemplar RAG from de-identified prior reports (see [[concepts/style-profiles]])
- **STYLE** — Quarto `neurotyp-*-typst` formats via [[concepts/quarto-extensions]]

This supports one of Luria's stated priorities: preserving a credible clinical voice rather than producing generic AI prose. The repository rebuild plan further implies that these assets are being separated into `voice_assets/` and related service modules, making voice and reporting concerns more explicit architectural components rather than incidental folders.

### Honcho AI (Optional)
[[concepts/honcho-ai-peer-observation]] enables peer-observation patterns via the Honcho SDK for session-based agent monitoring.

### R Integration
Optional R integration for statistical analysis and visualization using `dplyr`, `tidyr`, `ggplot2`, `psych`, and `reticulate`. See [[concepts/r-python-integration]], [[concepts/r-neuropsych-packages]], [[concepts/quarto]], and [[concepts/r-visualization-theming]].

### Database & Storage
Dual local stores: SQLite for relational data and [[concepts/lancedb-vector-store]] for semantic retrieval. Related: [[concepts/clinical-data-management]], [[concepts/retrieval-augmented-generation]].

### Workflow and QA Layers
The investor memo highlights several newer integration priorities that help define Luria's platform direction:
- score processing automation with clinical validity checks
- differential-diagnosis support in the interpretation engine
- quality assurance automation
- real-time clinician feedback loops
- Desktop Commander integration for seamless clinician workflow

These features reinforce Luria's identity as a workflow platform rather than a single drafting model.

---

## Installation & Distribution

Distributed as a standard Python package (Python 3.13+ required for the Streamlit app):
- **uv** (recommended): `uv sync`
- **pip**: `pip install -r requirements.txt`
- **From source**: `github.com/brainworkup/luria`

Primary platform is macOS (required for MacWhisper and local MLX inference). Optional: Quarto, Typst, MacWhisper CLI (`mw`), local [[concepts/omlx-server]].

See [[summaries/installation]] for full instructions.

## System Requirements

| Level | Python | RAM | Disk |
|---|---|---|---|
| Minimum | 3.10+ | 4GB | 1GB |
| Recommended | 3.13+ | 8GB+ | 2GB+ |

R integration additionally requires R 4.0+.

---

## Configuration

Luria uses a `.env` file for environment configuration. Key variables: `ANTHROPIC_API_KEY`, `OMLX_BASE_URL`, `OMLX_CHAT_MODEL`, `OMLX_EMBEDDING_MODEL`, `NEUROPSYCH_SOUL_DB`, `NEUROPSYCH_SOUL_PROFILE`. See [[concepts/security-policy]] and [[concepts/phi-data-handling]].

---

## KB Roadmap

- [x] Tier 1 wiki long-context retrieval with `kb ask` CLI and cited answers
- [ ] Tier 2 router with per-corpus embeddings dispatch
- [ ] Soul handoff — move PAI exemplars to `voice/soul`
- [ ] `shared_references/` restructuring into typed sub-corpora
- [ ] FastAPI service mode — wrap `kb ask` as a small endpoint
- [ ] OpenKB integration as persistent knowledge backend
- [ ] Citation rendering — convert returned spans into clickable wikilink references
- [ ] Expand Luria.app toward a fuller production version of the end-to-end clinical workflow described in [[summaries/Apply-to-Y-Combinator-JWT]]
- [ ] Complete migration from legacy monolithic folders into the emerging `app/`, `services/`, `api/`, `voice_assets/`, and `external/` layout documented in [[summaries/File Folder Structure Rebuild]]
- [ ] Rewrite imports and reconcile duplicated ingest and retrieval paths after the structural move
- [ ] Support enterprise deployment infrastructure and healthcare-system pilots described in [[summaries/Luria_AI_Q4_Investor_Memo_2026]]
- [ ] Complete compliance and validation milestones such as SOC 2 Type I readiness and multi-site clinical validation support

---

## Architectural Context

Luria sits at the intersection of several technical and clinical domains:
- [[concepts/neuropsychological-assessment-pipeline]] — the clinical workflow it supports
- [[concepts/luria-neuropsych-pipeline]] — the specific data pipeline architecture
- [[concepts/knowledge-base-architecture]] — the two-tier KB design pattern
- [[concepts/subagent-architecture]] — system prompts sourced from `subagents/*/AGENTS.md` files
- [[concepts/local-llm-inference]] — on-device inference via MLX
- [[concepts/mlx-framework]] — local Apple Silicon inference stack
- [[concepts/python-environment-management]] — recommended tooling
- [[concepts/python-project-structure]] — historical and evolving project organization
- [[concepts/narrative-report-generation]] — the report generation output layer
- [[concepts/neuropsychological-assessment-automation]] — broader automation context
- [[concepts/hybrid-search-retrieval]] — SQL + semantic search in the Ask tab
- [[concepts/audio-transcription-pipeline]] — the Audio tab pipeline
- [[concepts/clinical-ai-reasoning]] — Console & Synthesis reasoning engine
- [[concepts/neuropsychological-synthesis]] — AI-driven pattern detection and formulation
- [[concepts/clinical-ai-copilot]] — the conversational copilot pattern underlying the Console
- [[concepts/fallback-strategy]] — provider fallback chain design
- [[concepts/multi-agent-orchestration]] — section agent orchestration in report generation
- [[concepts/clinical-data-privacy]] — the privacy boundary governing deployment choices
- [[concepts/clinical-narrative-generation]] — clinician-grade integrated report prose
- [[concepts/neuropsychological-assessment-workflow]] — the broader end-to-end workflow vision
- [[concepts/forensic-neuropsychological-evaluation]] — one of the demanding use cases highlighted by the founder
- [[concepts/codebase-reorganization]] — the staged repository restructuring now underway
- [[concepts/migration-strategy]] — deterministic directory moves with legacy preservation
- [[concepts/repository-hygiene]] — preserving unknown files while separating active and experimental systems
- [[concepts/healthcare-workforce-automation]] — the market-level problem Luria aims to solve
- [[concepts/human-in-the-loop-clinical-ai]] — clinician feedback loops and supervised deployment
- [[concepts/healthcare-ai-regulation]] — compliance and regulatory pathways relevant to clinical scale

---

## Project Structure Highlights

Historically, Luria has been organized around a Python application, agent packages, and a knowledge base:

```text
kb/
├── wiki/                     ← Tier 1: curated markdown KB (~40 notes)
│   ├── _index.md
│   ├── topics/               ← 9 pillar pages
│   └── notes/                ← 31 concept notes
├── store/                    ← Tier 2: specialized sub-corpora (vendored)
│   ├── typst/
│   ├── quarto/
│   └── shared_references/
├── kb_ask.py                 ← Tier-1 CLI (long-context retrieval)
├── pyproject.toml
└── README.md
```

Within Luria itself, the legacy structure includes:
- `neuropsych_agent/` — LangGraph orchestration (graph, nodes, state, tools)
- `neuropsych_rag/` — Standalone reusable RAG library
- `subagents/` — AGENTS.md prompt libraries for each pipeline stage
- `skills/` — Reusable agent skills; see [[concepts/luria-skills]]
- `data/` — Runtime data directory (gitignored): uploads, reports, SQLite DB, vectors

The emerging structure documented in [[summaries/File Folder Structure Rebuild]] clarifies the intended future organization:
- `app/streamlit/` — UI and pages
- `services/ingest`, `services/retrieval`, `services/agent`, `services/storage`, `services/voice`, `services/reporting`, `services/integrations`
- `api/fastapi/routes/` — API layer
- `voice_assets/` — brand, style, soul resources
- `external/` — archived or legacy systems such as cingulate and experimental RAG code

This contrast between current and target structure is useful for interpreting the rest of the documentation: many existing materials describe the working legacy layout, while newer materials describe the modular architecture the project is moving toward. The investor memo adds the practical next stage for that structure: first healthcare-system pilots, enterprise deployment infrastructure, and team growth oriented toward implementation and validation.

---

## Business and Validation Snapshot

The Q4 2026 investor memo ([[summaries/Luria_AI_Q4_Investor_Memo_2026]]) provides the clearest concise snapshot of Luria's current business and validation status:

- **Active cases processed:** 47 in Q4 2026, up from 12 in Q3
- **Report quality:** 94% clinician-validated accuracy
- **Time savings:** 89% reduction per case (8.5 hours → 55 minutes)
- **Pipeline:** 3 healthcare-system pilots in negotiation
- **Revenue:** pre-revenue, with focus on clinical validation

The memo also describes near-term goals for Q1 2027:
- launch first healthcare-system pilot at 500+ cases/month
- complete SOC 2 Type I certification
- begin multi-site clinical validation study
- hire first clinical implementation specialist

Its planned monetization model includes:
- **Per-case licensing:** $25–50 per case
- **System licensing:** $50k–250k annually per healthcare system

This matters for the concept of Luria because it confirms that the project is not only an evolving technical stack or design prototype. It is also becoming a clinically validated, enterprise-oriented product with a defined deployment, compliance, and fundraising roadmap. The memo's seed-round ask and regulatory language also suggest that future versions of Luria may need to satisfy more formal quality, security, and clinical evidence standards than the current documentation alone would imply.

---

## Related Pages

- [[summaries/installation]] — Step-by-step installation and configuration guide
- [[summaries/README_luria]] — Project README overview
- [[summaries/README]] — Luria KB README
- [[summaries/overview]] — High-level project overview
- [[summaries/AGENTS_luria]] — Agent architecture within Luria
- [[summaries/full-pipeline]] — Full pipeline documentation
- [[summaries/KNOWLEDGE_BASE_EXPLAINED]] — KB architecture explanation
- [[summaries/redesign_20260623110817]] — 2026 Luria.app UI redesign documentation (Part 1)
- [[summaries/redesign_20260623110910]] — 2026 Luria.app UI redesign documentation (Part 2: full page specs and OpenKB integration)
- [[summaries/Apply-to-Y-Combinator-JWT]] — Founder narrative and YC framing for Luria's product vision
- [[summaries/File Folder Structure Rebuild]] — Safe repository migration plan for the emerging modular architecture
- [[summaries/Luria_AI_Q4_Investor_Memo_2026]] — Q4 2026 investor update on traction, validation, and scaling plan

See also: [[summaries/LLM Benchmark Comparison]]

See also: [[summaries/README_20260413235353]]

See also: [[summaries/README_20260413235533]]