---
sources: [summaries/clinical-assessment.md, summaries/README_20260413235353.md, summaries/Luria_AI_Q4_Investor_Memo_2026.md, summaries/Apply-to-Y-Combinator-JWT.md, summaries/redesign_20260623110910.md, summaries/redesign_20260623110817.md, summaries/SESSION_SUMMARY.md, summaries/QUICK_REFERENCE.md, summaries/PERMANENT_SOLUTION_SUMMARY.md, summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_PIPELINE.md, summaries/DIAGNOSIS_FIX_SUMMARY.md, summaries/AUTISM_RAG_SYSTEM_DOCUMENTATION.md, summaries/AGE_OVERRIDE_GUIDE.md, summaries/nse_narrative.md, summaries/agent-team.md, summaries/DEPENDENCIES.md, summaries/2026-04-26-cingulate-agent-team-design.md, summaries/0009-soul-local-llm-inference-with-omlx.md, summaries/neuropsych-pdf-parser.md, summaries/neuropsych-narrative-writer.md, summaries/neuropsych-data-extractor.md, summaries/clinical-validity-reviewer.md, summaries/processed_files.md, summaries/TECHNICAL_DOCS.md, summaries/README_WORKFLOW.md, summaries/README_PIPELINE.md, summaries/AS_PROCESSING_COMPLETE.md, summaries/brainworkup-brand-voice-guide.md, summaries/mcp-integration.md, summaries/002-mcp-llm-integration.md, summaries/SKILL.md, summaries/README.md, summaries/project-setup-progress.md, summaries/AGENTS_luria.md, summaries/README_luria.md, summaries/deepagents_merged_mem_notes.md, summaries/SETUP_SUMMARY.md, summaries/SESSION_SUMMARY_2025-04-28.md, summaries/RECOVERY_NOTES.md]
brief: How clinical systems prevent PHI exposure across storage, agents, and reporting.
---

# PHI Data Handling in Clinical Software

Protected Health Information (PHI) is any data that can identify a patient and relates to their health, treatment, or payment history. In clinical software development, PHI handling is governed by regulations such as HIPAA in the United States, which impose strict requirements on storage, transmission, and access control. For developers, one of the most common and consequential failure modes is **accidentally committing PHI to version control** — a mistake that is difficult to remediate and potentially carries legal liability.

In neuropsychology and adjacent clinical domains, PHI handling is not a peripheral compliance task; it is a core architectural concern. The founder narrative in [[summaries/Apply-to-Y-Combinator-JWT]] makes this especially clear: the product vision for Luria is built around automating the full neuropsychological evaluation workflow while preserving the non-negotiable constraints of **clinical voice**, **cross-domain synthesis**, and **strict PHI boundaries**. That framing is important because it treats privacy not as a deployment option but as a product requirement. This aligns closely with [[concepts/privacy-first-software]], [[concepts/local-first-architecture]], and [[concepts/neuropsychological-assessment-automation]].

## Why Version Control Is a High-Risk Surface

Development workflows that mix source code with runtime data directories create opportunities for PHI to enter git history. Once committed, data is persistent across all clones and forks of the repository, and removing it requires history rewriting, which disrupts collaborators and may still leave traces in local caches. The safest approach is **prevention at the boundary**: ensure that directories containing runtime data are excluded from tracking before any data ever lands in them.

The cingulate agent team design (see [[summaries/2026-04-26-cingulate-agent-team-design]]) makes this an explicit pre-flight requirement: `output/` — the per-patient workspace root that holds all stage outputs including rendered PDFs and QA issue lists — must be confirmed gitignored before any real case run.

This concern becomes even more important in products like Luria, whose stated goal is to automate an end-to-end neuropsychological workflow and to ingest material from many source types. In [[summaries/Apply-to-Y-Combinator-JWT]], the founder describes an evolution from CSV extraction to PDF handling and eventually to processing arbitrary file types. As the number of ingestion surfaces grows, so does the number of places where PHI can accidentally be cached, logged, staged, or committed. Version control therefore becomes a primary containment boundary, not just a developer convenience.

## Key Practices

### 1. Aggressive `.gitignore` Configuration

Runtime data directories should be listed in `.gitignore` as early as possible in a project's lifecycle — ideally before any data is generated. Patterns should cover:

- **Upload staging areas** — directories where raw patient files (PDFs, scans) are deposited.
- **Processed output directories** — directories containing extracted text, structured JSON, generated reports, or rendered PDFs.
- **Database files** — SQLite databases (`data/neuropsych.db`), vector store indexes (LanceDB at `data/vectors/`), the Soul style exemplar SQLite index (`report_soul_index.sqlite`), or embedded databases that accumulate patient records at runtime.
- **Per-patient workspace trees** — in the cingulate architecture, the entire `output/<patient_slug>/` tree.
- **OS and editor artifacts** — `.DS_Store`, `.history/`, IDE workspace files.

In the Luria repository recovery documented in [[summaries/RECOVERY_NOTES]], this step was explicitly addressed: `.DS_Store`, `**/.DS_Store`, `.history/`, and all runtime-data directories under `app/streamlit/data/` were added to `.gitignore`.

The Luria Streamlit App (see [[summaries/README]]) reinforces this: `data/uploads/`, `data/reports/`, `data/neuropsych.db`, and `data/vectors/` are all gitignored by default, concentrating all runtime PHI-bearing artifacts under the `data/` subtree.

For products intended to support full clinical workflows, `.gitignore` design should be treated as part of the privacy model. If a system is meant to automate report writing, document ingestion, test-score extraction, and synthesis across domains, then every intermediate artifact those stages produce should be assumed PHI-bearing unless explicitly proven otherwise.

### 2. Separating Code from Data at the Filesystem Level

A clean workspace layout separates source code from runtime data by convention and enforces this with tooling. In practice this means:

- Code lives under version-controlled directories (`app/`, `scripts/`, `subagents/`, `.claude/agents/`).
- Runtime data lives under untracked directories (`data/uploads/`, `data/reports/`, `output/<patient_slug>/`).
- Configuration files that might contain credentials or patient identifiers are excluded or stored in environment variables (`.env` — always gitignored, never committed).

The Luria dependency manifest formalizes this at the storage backend level. Four distinct backends are declared, each with a specific untracked path:

| Backend | Path | Content |
|---|---|---|
| SQLite | `data/neuropsych.db` | Structured test scores + summaries |
| LanceDB | `data/vectors/` | Semantic search vectors |
| SQLite (Soul) | `report_soul_index.sqlite` | Style exemplar vectors |
| DuckDB | `kb/store/` | Analytics / RAG column-store |

All four paths must be covered by `.gitignore`. The cingulate agent team design formalizes the code/data separation architecturally: the per-patient workspace (`output/<patient_slug>/`) is the sole inter-stage contract, meaning all PHI-bearing artifacts are concentrated in a single, easily gitignored subtree.

The Luria refactor documented in [[summaries/SESSION_SUMMARY_2025-04-28]] reinforces this principle: patient data in a SQLite store and vector indexes sit outside the tracked source tree, while PII obfuscation is handled by a dedicated service.

The founder account in [[summaries/Apply-to-Y-Combinator-JWT]] adds useful product-level context here. Because the target workflow is the full neuropsychological evaluation process — including integration of test batteries, interviews, and observations into final reports — the filesystem layout must assume that nearly every runtime artifact may be sensitive. A strong code/data split is therefore part of making a system viable for neuropsychological use, not just part of keeping a repository tidy.

### 3. The PII Redaction Gate Pattern

Beyond `.gitignore` hygiene, a well-designed clinical pipeline should include an active PII redaction step before data is indexed, stored, or passed to any subagent. The corrected Luria orchestration plan establishes a formal **PII redaction gate** implemented via Microsoft Presidio (`tools/pii_presidio.py`). This gate:

- Detects and masks names, dates of birth, addresses, phone numbers, MRNs, and SSNs in the raw NSE transcript.
- Produces a single redacted output (`nse_summary_redacted.md`) that all downstream agents consume.
- Ensures that even if a downstream agent were compromised or logged, it would never have seen raw identifiers.

The Luria app redesign (see [[summaries/redesign_20260623110910]]) demonstrates this pattern in production form. The **Console & Synthesis** module (⌘4) shows how the AI copilot reasons over clinical data while every claim is cited back to a grounded source in the evidence rail. Crucially, the app's LLM infrastructure enforces a **local-only gate** via `restrictToPreferredProviders: true`, which ensures PHI never reaches cloud providers: the fallback chain prioritizes oMLX → vMLX (Responses API) → Ollama, and only falls back to cloud endpoints for requests explicitly marked as non-PHI. The `redactPhi()` function is the first node in the `/stt/summarize` pipeline, and the `LocalFallbackLLMClient` enforces `restrictToPreferredProviders: true` on every `.generate()` call that carries patient data.

The Luria Streamlit App implements a complementary architectural constraint: PHI redaction happens locally via Docling's parse stage **before** any text is sent to Anthropic's Claude API. Only after local redaction does the extraction step call the cloud API. The `PHI_REDACTION_ENABLED=true` environment flag controls this gate; when `LLM_PROVIDER=omlx`, the entire pipeline runs without any cloud API call at all, making PHI exposure through external services architecturally impossible. [[concepts/docling-pdf-parsing]] handles structural extraction from raw PDFs, and Presidio masks identifiers in the resulting text before it enters any LLM context.

This represents a defense-in-depth architecture: the **passive boundary control** (`.gitignore`, filesystem permissions) prevents PHI from entering version control, while the **active redaction gate** (Presidio / `redactPhi()`) prevents PHI from propagating through the agent graph.

The cingulate agent team design reinforces this at a structural level. The `cingulate-quality-reviewer` subagent — the final stage in the pipeline — performs a PHI scan as part of its review before surfacing the draft report.

The founder narrative in [[summaries/Apply-to-Y-Combinator-JWT]] strengthens the rationale for this pattern: neuropsychological reports are described as the main clinical deliverable and the most labor-intensive part of the evaluation. Because the product's value comes from integrating many sensitive inputs into one high-value report, the redaction gate must sit upstream of all synthesis components. In other words, the more capable the narrative engine becomes, the more important it is that it receives only appropriately sanitized context.

### 4. Anonymization Tokens at Every Pipeline Stage

PHI protection must extend into every artifact a clinical pipeline produces — not just intermediate data but also the final output reports. The PDF Ingestion & Parser Worker (the first stage of the [[concepts/neuropsychological-assessment-pipeline]], documented in [[summaries/AGENTS_luria]]) enforces this from the very beginning: whenever PHI is encountered during document parsing, patient names are replaced with `[PATIENT_ID]` and clinician names with `[CLINICIAN]` before the structured output is passed downstream.

This anonymization-at-ingestion principle applies equally to stage 2 of the pipeline. The `neuropsych-data-extractor` — the structured score extraction agent that converts parsed text into cingulate-format CSV rows — carries an explicit PHI rule: if any real names or identifiers slip through from the parser, they must be replaced with `[PATIENT]` or `[CLINICIAN]` immediately, and a WARNING must be logged in the agent's output summary block.

The same token convention carries through to the Narrative Report Generator stage and — critically — to the Typst report formatter skill (documented in [[summaries/SKILL]]). The Typst output skill enforces identical anonymization rules: the `patient:` field in the Typst template must always use the `[PATIENT_ID]` token, never a real patient name, and `[CLINICIAN]` must be substituted for any evaluator reference outside the fixed `Provider:` field.

In the redesigned Luria app, the **Report Builder** (⌘7) drafts from 6 tests and 3 sources with citations enabled, and the active case-file sidebar always displays only the patient alias rather than any real identifier. The `agentRunner.ts` agent context passes only tokenized clinical data and patient scores to section agents, not raw identifiers.

See [[concepts/redaction-tokens]] for the token conventions enforced across all pipeline stages and output formats.

### 5. PHI Verification in Pre-Sign-Out Review

A dedicated review agent — the clinical-validity-reviewer — adds an explicit PHI verification axis to the neuropsychological report pipeline. Before a report is signed out, this read-only agent runs grep-based scans across all draft `_text.qmd` narrative files, searching for:

- Real patient or clinician names (anything not `[PATIENT]`, `[CLINICIAN]`, or `[FACILITY]`).
- DOB-shaped strings: `(0[1-9]|1[0-2])/[0-3]?\d/(19|20)\d{2}`.
- SSN-shaped strings: `\d{3}-\d{2}-\d{4}`.
- MRN-shaped strings: ≥6 consecutive digits not clearly attributable to a score.
- Phone number patterns.
- Literal addresses, email addresses, and facility names.

Any finding at this axis triggers a `CRITICAL_ISSUES` classification that **blocks sign-out** — the highest severity level in the reviewer's verdict system.

See also [[concepts/report-review-qa]] for the broader review and quality-assurance pattern.

This final verification step matters especially in settings like those described in [[summaries/Apply-to-Y-Combinator-JWT]], where long-form integrated interpretation is the core output. Neuropsychological synthesis often reconstructs a patient story from many fragments; even when direct identifiers have been removed, generated prose can accidentally reintroduce identifying details through context, chronology, or distinctive combinations of facts. A pre-sign-out PHI scan is therefore a clinical safety mechanism as much as a privacy mechanism.

### 6. HIPAA-Aware Permission Models in Multi-Agent Systems

When clinical pipelines are implemented as [[concepts/multi-agent-orchestration]] systems, PHI access control must extend to the agent permission model. The corrected Luria orchestration plan enforces this explicitly:

- Subagents are **denied access** to `data-raw/` (raw PHI) and `reports/final/` (requires human approval).
- Only the redaction gate agent reads from the raw transcript; all other agents consume only redacted content.
- Human-in-the-loop approval is required before any report leaves the system.

The cingulate agent team design applies this same principle through the [[concepts/subagent-architecture]] pattern. Each stage subagent receives only its relevant context slice — never the full conversation history. The dispatch flow is strictly sequential (intake → scoring → interpretation → report → qa), meaning downstream agents never have access to raw intake data.

The redesigned Luria app formalizes this at the TypeScript layer. The `llmAbortContext` uses `AsyncLocalStorage` to thread an abort signal through `generate()`, enabling mid-fallback job cancellation without leaking partial PHI-bearing state. The `reportJobs.ts` orchestrator coordinates `createReportJob()` and `runReportJob()` in sequence, and each section agent in `agentRunner.ts` receives only the `AgentContext` slice relevant to its domain.

The `cingulate-orchestrator` manages `state.json` as the sole coordination mechanism. Because `state.json` contains only status fields and timestamps, the orchestrator itself never needs access to patient data.

The Luria dependency manifest reinforces this at the framework level: the oMLX local server (`LLM_PROVIDER=omlx`) enables all chat and embeddings to run without internet, and the `openai` package is used only to call oMLX's OpenAI-compatible `/v1` endpoint on localhost — not the OpenAI cloud service. The `ANTHROPIC_API_KEY` for Claude fallback is stored in `.env` (gitignored), and the manifest notes explicitly: **PHI is redacted before any call** to Anthropic.

The product framing in [[summaries/Apply-to-Y-Combinator-JWT]] is notable here because it explicitly describes Luria as a team of agents and subagents that can execute a clinical workflow nearly autonomously. That kind of capability increases the importance of least-privilege design. The stronger the agentic workflow, the more carefully PHI access must be partitioned so that autonomy does not imply broad data exposure.

### 7. Dedicated PII Obfuscation Layer

In the original Luria architecture, the `rag/docling/` service provides obfuscation on documents during ingestion. This means that even if downstream storage were accidentally exposed, the indexed content would not contain raw identifiers. The corrected orchestration plan adds Presidio as a second, more targeted redaction tool specifically for the NSE transcript pipeline, giving the system two complementary obfuscation mechanisms with different strengths: [[concepts/docling-pdf-parsing]] for structural document parsing and Presidio for named-entity recognition and masking.

The redesigned app adds a third mechanism: the `redactPhi()` TypeScript function embedded in the `/stt/summarize` route, which sits between audio transcription input and LLM inference. This three-layer approach (docling structural parsing → Presidio NER masking → `redactPhi()` runtime gate) provides defense in depth at the document, entity, and runtime levels respectively.

The Luria Streamlit App reinforces this layered approach: its `neuropsych_rag/ingest/` module includes dedicated redaction and chunking components, while the LangGraph pipeline (built on `langgraph`) enforces redaction as a prerequisite to any cloud API call. See [[concepts/langgraph-agent-workflows]] for the broader pipeline architecture.

This layered design matches the founder's claim in [[summaries/Apply-to-Y-Combinator-JWT]] that generic cloud-based AI is not sufficient for this domain. The implication is architectural: PHI handling in neuropsychology requires multiple privacy controls working together, not a single policy statement about compliance.

### 8. Quality Review as a PHI Verification Step

The corrected Luria plan includes a dedicated quality review phase whose checklist explicitly includes a **Presidio re-scan** of the assembled report narrative for any PHI that may have leaked through the generation pipeline. This is a critical safeguard: language models can sometimes reconstruct identifiers from context even when inputs were redacted.

In the cingulate agent team design, `cingulate-quality-reviewer` performs the equivalent function. It reads the rendered PDF and all upstream outputs, then writes findings to `qa/issue_list.md`. Because this stage runs after `quarto render` produces the final PDF, it catches PHI that may have been introduced at the Quarto template assembly or Typst rendering stages.

See also [[summaries/clinical-validity-reviewer]] for the read-only pre-sign-out review agent specification.

In the workflow described in [[summaries/Apply-to-Y-Combinator-JWT]], where a final report is the clinical product delivered to families, patients, and other providers, this QA step is especially high stakes. A PHI leak in an intermediate file is serious; a leak in the final signed report is a production failure.

### 9. Smoke Testing Path Resolution

A common indirect PHI risk is a misconfigured path that causes the application to write data to an unexpected — and potentially tracked — location. The Luria recovery introduced `scripts/smoke_test_paths.py` specifically to verify that every static path the pipeline relies on resolves correctly before a demo or deployment.

The cingulate agent team design addresses the analogous risk through the CWD constraint documentation. Several cingulate helpers expect `setwd(workspace_path)` after `devtools::load_all()`. If this `setwd` call is omitted or misconfigured, helpers may write outputs to the repository root or another tracked location, inadvertently capturing PHI.

As systems evolve from narrow scripts to broad workflow platforms, this class of error becomes more likely. The founder account in [[summaries/Apply-to-Y-Combinator-JWT]] describes a progression from small R utilities to a largely autonomous multi-agent workflow. That kind of growth typically increases the number of implicit path assumptions in the codebase, which makes smoke testing path boundaries a practical PHI safeguard.

### 10. Local-Only Processing

For clinical software, running inference and data processing entirely on local hardware eliminates a major PHI exposure vector. This connects directly to [[concepts/local-llm-inference]] and [[concepts/privacy-first-software]], both of which emphasize keeping sensitive data within a controlled boundary.

The redesigned Luria app makes this the architectural default. The LLM infrastructure (`src/services/llmClient.ts`) defines a `LocalFallbackLLMClient` with a four-provider fallback chain:

| Priority | Provider | PHI-safe? |
|---|---|---|
| 1 | oMLX | ✓ Local |
| 2 | vMLX (Responses API) | ✓ Local |
| 3 | Ollama | ✓ Local |
| 4 | Cloud | ✗ Non-PHI only |

The `restrictToPreferredProviders: true` flag enforces the local-only gate. Cloud endpoints are reachable **only** when a request carries no PHI. This makes the PHI boundary architectural rather than policy-dependent — it cannot be bypassed by a misconfigured environment variable or an accidental call without the flag.

The Luria Streamlit App makes this architecture explicit and is designed as a local-first application from the ground up (see [[summaries/README]]):

- PDF parsing (Docling), embeddings, vector search ([[concepts/lancedb-vector-store]]), and structured storage (SQLite) all run on the local machine.
- Only the extraction step optionally calls Anthropic's API — and only after PHI redaction.
- Audio transcription runs locally; summarization via the oMLX server also runs locally.
- The `openai` Python package is used only to reach the local oMLX `/v1` endpoint — not the OpenAI cloud.
- When `LLM_PROVIDER=omlx`, the entire pipeline operates without any external API call, making PHI exposure through external services architecturally impossible.

The Luria dependency manifest makes the same architecture explicit:

- **oMLX** runs the default chat/extraction model locally — no cloud required.
- **Embeddings** default to a local model, with a second local fallback.
- **LanceDB** stores all semantic search vectors on local disk (`data/vectors/`).
- **SQLite** stores all structured test scores locally (`data/neuropsych.db`).

See [[concepts/local-first-architecture]] for the broader design philosophy, and [[concepts/omlx-server]] for the local LLM inference configuration.

The founder description in [[summaries/Apply-to-Y-Combinator-JWT]] directly supports this section: one of the product's stated differentiators is that it can handle the sensitive nature of neuropsychological evaluations by keeping all data local and secure. In this framing, local-only processing is not merely a compliance tactic; it is part of the product's competitive advantage.

### 11. API Key Hygiene as a PHI-Adjacent Risk

API keys committed to version control can enable unauthorized access to LLM APIs that may have processed PHI-adjacent data. The Luria Streamlit App README explicitly notes: "Rotate any API keys that may have been committed before `.gitignore` was finalized." The `.env.example` template pattern ensures that actual keys are never committed.

The Luria dependency manifest documents three API key surfaces:

- `ANTHROPIC_API_KEY` — Claude extraction fallback (cloud)
- `OPENAI_API_KEY` — GPT fallback (optional, cloud)
- `LANGSMITH_API_KEY` — tracing

All three are `.env`-gated. The `OMLX_BASE_URL` configuration key for the local oMLX server requires no secret — it is a localhost URL — making the default configuration inherently safer than any cloud-API-dependent alternative.

For privacy-first clinical systems, secret hygiene should be treated as part of PHI boundary management. Even if keys do not directly reveal patient data, they can enable access to systems that process or store PHI-bearing artifacts.

### 12. Audit and Verification Before Every Sensitive Push

Before pushing to a remote — especially before a demo, external review, or funding conversation — developers should:

- Run `git status` and `git diff --cached` to confirm no data files are staged.
- Run the smoke test to confirm path resolution is correct.
- Review `.gitignore` for any recently added data directories that may not yet be excluded.
- Confirm `output/` is gitignored before any patient workspace is created under it.
- Verify all four storage backends (`data/neuropsych.db`, `data/vectors/`, `report_soul_index.sqlite`, `kb/store/`) are covered by `.gitignore`.
- Confirm that any demo data is synthetic, tokenized, or otherwise cleared for disclosure.

This last point is especially relevant to systems like the one described in [[summaries/Apply-to-Y-Combinator-JWT]], where founder demos may center on clinically realistic end-to-end workflow automation. The more compelling the demo, the greater the temptation to use realistic case material; audit discipline is what keeps that from becoming a PHI incident.

## PHI Handling Across Report Output Formats

The [[concepts/forensic-neuropsychological-evaluation]] context adds an important dimension: final reports may be produced in multiple formats and may be submitted as medicolegal work product. The anonymization token convention (`[PATIENT_ID]`, `[CLINICIAN]`, `[CASE_NUMBER]`) must therefore be consistently applied across all output formats and verified before any report artifact leaves the controlled pipeline.

The [[concepts/typst-typesetting]] system used for print-ready report output parameterizes PHI fields at the template level, making it structurally difficult to omit anonymization — the template will not render correctly without the required token fields. Quarto and Typst are declared as required external CLI tools in the Luria dependency manifest, and their role in the report rendering pipeline means the anonymization token layer must be enforced in any Quarto/Typst template added to the workspace.

The redesigned Luria Report Builder (⌘7) illustrates this in practice: draft sections are generated with citations enabled, and voice/reading-level controls allow output tuned for clinical, balanced, or parent audiences — but the PHI token layer is invariant across all voice settings.

This invariance matters because, as noted in [[summaries/Apply-to-Y-Combinator-JWT]], the final report is the core deliverable in neuropsychological practice. If the product supports multiple reporting styles or tones, privacy constraints must remain stable across all of them.

## PHI Containment in the Structured Score Layer

The cingulate long-format CSV schema used by the `neuropsych-data-extractor` represents a distinct PHI surface that warrants specific attention. The schema (see [[concepts/long-format-clinical-data]]) includes a `doc_id` column and a `date` column. Neither field directly identifies a patient by name, but the combination of a specific test date and score profile could constitute indirect PHI in some regulatory interpretations.

The `neuropsych-data-extractor`'s design mitigates PHI risk in two ways:

1. **Schema-level exclusion**: The canonical schema contains no name, address, MRN, or contact-information column. PHI cannot be stored in a field that does not exist.
2. **Upstream gate dependency**: The agent is explicitly designed to receive text that has already been scrubbed by the pdf-parser stage. If identifiers appear in the input, they are treated as a gate failure and flagged as warnings — triggering review rather than silent propagation.

In the redesigned app, the structured data layer is surfaced through the RAG-backed document index as discrete, cited claims rather than raw document text, reducing accidental exposure at the UI layer.

This is important for end-to-end neuropsychological automation because integrated interpretation depends heavily on structured scores, timelines, and cross-test relationships. The more central the structured layer becomes to synthesis, the more valuable schema-level PHI minimization becomes.

## Relationship to Clinical NLP Pipelines

PHI handling constraints shape the architecture of [[concepts/clinical-nlp-pipelines]] at every layer. Ingestion, extraction, indexing, and report generation all touch patient data, and each stage must be audited for logging behavior, temporary file creation, and error output — all of which can inadvertently capture PHI if not carefully managed.

The cingulate agent team design reinforces this with a specific logging convention: each stage writes to `logs/<stage>.log`, and absolute paths are required in all logs and `state.json`. This makes logs auditable — but it also means logs must be treated as potentially sensitive artifacts and included in the gitignore scope.

The domain fan-out architecture used in the corrected Luria plan illustrates how PHI containment can be maintained even at high concurrency: by ensuring the redaction gate runs before the fan-out, all parallel agents operate on pre-sanitized data, and PHI risk does not scale with concurrency.

The redesigned Luria Console (⌘4) makes this architecture user-visible: the evidence rail displays cited sources with score values but never raw transcript text. The local model indicator in the console toolbar signals to the clinician that inference is running locally, reinforcing the privacy guarantee at the UI level.

The founder story in [[summaries/Apply-to-Y-Combinator-JWT]] adds a domain reason for all of this complexity: neuropsychological evaluation requires integrating interviews, observations, and many test results into a coherent narrative that spans neurocognitive and neurobehavioral domains. That is exactly the kind of clinical NLP problem where PHI handling cannot be bolted on later. The privacy model has to be embedded in ingestion, storage, orchestration, synthesis, and output from the beginning.

## See Also

- [[summaries/Apply-to-Y-Combinator-JWT]] — Founder narrative describing Luria as a local-first, agent-based neuropsychology workflow system built around strict PHI boundaries.
- [[summaries/2026-04-26-cingulate-agent-team-design]] — Cingulate agent team scaffolding design, including the per-patient workspace structure and the gitignore pre-flight requirement.
- [[summaries/RECOVERY_NOTES]] — Recovery session that prompted the `.gitignore` hardening described above.
- [[summaries/SESSION_SUMMARY_2025-04-28]] — Architecture refactor session introducing explicit PII obfuscation service and SQLite-based patient data store.
- [[summaries/deepagents_merged_mem_notes]] — Corrected Luria orchestration plan establishing the Presidio redaction gate and subagent PHI permission model.
- [[summaries/AGENTS_luria]] — PDF Ingestion & Parser Worker specification, including anonymization token requirements.
- [[summaries/neuropsych-data-extractor]] — Stage 2 score extraction agent, with PHI slip-through detection and warning-based remediation.
- [[summaries/SKILL]] — Typst Report Formatter skill, specifying PHI anonymization rules for print-ready `.typ` report output.
- [[summaries/clinical-validity-reviewer]] — The read-only pre-sign-out review agent that includes a grep-based PHI leak detection axis as a hard sign-out blocker.
- [[summaries/README]] — Luria Streamlit App README documenting local-first PHI architecture, gitignored data directories, and cloud-API-only-after-redaction design.
- [[summaries/DEPENDENCIES]] — Full dependency manifest specifying local storage backends, oMLX local LLM configuration, and PHI-relevant environment variables.
- [[summaries/redesign_20260623110910]] — Luria app redesign showing PHI-safe LLM infrastructure with `restrictToPreferredProviders` local-only gate and `redactPhi()` runtime guard.
- [[concepts/pii-redaction-pipelines]] — Broader treatment of PII detection and masking pipeline patterns.
- [[concepts/privacy-first-software]] — Broader principle of building software that minimizes data exposure by design.
- [[concepts/local-llm-inference]] — Running models locally to avoid transmitting PHI to external services.
- [[concepts/local-first-architecture]] — Design philosophy prioritizing local storage and processing for sensitive data.
- [[concepts/clinical-nlp-pipelines]] — Pipelines that process PHI at every stage and require careful boundary enforcement.
- [[concepts/neuropsychological-reporting]] — Domain context in which PHI handling requirements are especially stringent.
- [[concepts/multi-agent-orchestration]] — Orchestration patterns that must extend least-privilege access control to individual agents.
- [[concepts/subagent-architecture]] — The subagent dispatch pattern where each agent receives only its necessary context slice.
- [[summaries/README_luria]] — Luria project overview providing broader context for the PHI architecture described here.
- [[concepts/neuropsychological-assessment-pipeline]] — The pipeline within which the PDF Ingestion & Parser Worker enforces first-stage anonymization.
- [[concepts/forensic-neuropsychological-evaluation]] — Domain context requiring consistent anonymization across all report output formats.
- [[concepts/typst-typesetting]] — The typesetting system used for print-ready report output, with PHI fields parameterized at the template level.
- [[concepts/narrative-report-generation]] — Report generation stage that must apply anonymization tokens throughout generated prose.
- [[concepts/report-review-qa]] — The broader review and quality-assurance pattern that encompasses pre-sign-out PHI checks.
- [[concepts/redaction-tokens]] — Token conventions enforced across all pipeline stages and output formats.
- [[concepts/long-format-clinical-data]] — The structured CSV schema used by the score extraction stage, which is architecturally PHI-resistant by column design.
- [[concepts/docling-pdf-parsing]] — Local PDF parsing used as the first PHI redaction layer before any cloud API call.
- [[concepts/lancedb-vector-store]] — Local vector store used to avoid transmitting embedded patient data to cloud services.
- [[concepts/langgraph-agent-workflows]] — The pipeline engine that enforces parse → extract → index → report ordering with redaction gates.
- [[concepts/omlx-server]] — Local LLM inference server providing a fully offline alternative to cloud API calls.
- [[concepts/llm-provider-abstraction]] — The pattern of abstracting LLM provider selection behind a fallback chain with PHI-aware routing.
- [[summaries/neuropsych-narrative-writer]] — Narrative writer stage where LLM-generated prose is the highest-risk PHI reconstruction surface.
- [[summaries/neuropsych-pdf-parser]] — PDF parser stage specification with first-gate anonymization requirements.
- [[summaries/0009-soul-local-llm-inference-with-omlx]] — Local LLM inference architecture relevant to PHI-safe processing.
- [[summaries/README_PIPELINE]] — Pipeline architecture overview.
- [[summaries/TECHNICAL_DOCS]] — Technical documentation for the system.
- [[summaries/agent-team]] — Agent team design context.
- [[summaries/nse_narrative]] — NSE narrative pipeline context.
- [[summaries/AGE_OVERRIDE_GUIDE]]
- [[summaries/AUTISM_RAG_SYSTEM_DOCUMENTATION]]
- [[summaries/DIAGNOSIS_FIX_SUMMARY]]
- [[summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_PIPELINE]]
- [[summaries/PERMANENT_SOLUTION_SUMMARY]]
- [[summaries/QUICK_REFERENCE]]
- [[summaries/SESSION_SUMMARY]]
- [[summaries/redesign_20260623110817]]

See also: [[summaries/Luria_AI_Q4_Investor_Memo_2026]]

See also: [[summaries/README_20260413235353]]

See also: [[summaries/clinical-assessment]]