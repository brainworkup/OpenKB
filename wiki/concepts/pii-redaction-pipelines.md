---
sources: [summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_PIPELINE.md, summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_CODEMAP.md, summaries/AUTISM_RAG_SYSTEM_DOCUMENTATION.md, summaries/nse_narrative.md, summaries/README.md, summaries/SESSION_SUMMARY_2025-04-28.md, summaries/neuropsych-pdf-parser.md, summaries/neuropsych-data-extractor.md, summaries/clinical-validity-reviewer.md, summaries/local_models.md, summaries/mcp-integration.md, summaries/002-mcp-llm-integration.md, summaries/SKILL.md, summaries/AGENTS_luria.md, summaries/deepagents_merged_mem_notes.md]
brief: Layered PHI/PII redaction as a structural pipeline gate in clinical AI neuropsychological workflows.
---

# PII Redaction in Clinical AI Pipelines

PII (Personally Identifiable Information) and PHI (Protected Health Information) redaction is a critical gatekeeping step in any clinical AI pipeline that processes patient data. In neuropsychological and clinical reporting workflows, raw intake documents — transcripts, referral forms, audio recordings, prior records — contain highly sensitive identifiers that must be masked before any LLM subagent can process or reason over the content.

## Why Redaction Must Be a Hard Gate

In a multi-agent architecture, individual subagents cannot be trusted to spontaneously avoid quoting or retaining PII. The only reliable approach is to intercept content at a defined stage and strip identifiers *before* they reach any language model. This creates a clear trust boundary:

```
Raw documents (contains PHI)
        ↓
  [Redaction Gate]
        ↓
Redacted content (safe for LLM subagents)
```

All downstream agents receive only the redacted version. The raw data is never passed forward in the pipeline. This principle applies whether the pipeline is a sophisticated multi-agent system like the Luria neuropsychological pipeline or a streamlined local-first application like the Luria Streamlit app.

## Redaction in Local-First Desktop Applications

The Luria Streamlit App (see [[summaries/README]]) demonstrates that the [[concepts/local-first-architecture]] pattern is itself a powerful complement to software-level redaction. Key design decisions include:

- **PHI redaction happens locally** — the Docling parse stage strips identifiers on the user's own machine before any text is sent to the Anthropic API (Claude Sonnet).
- **No cloud vector store** — [[concepts/lancedb-vector-store]] and SQLite are entirely local, so redacted and structured data never leaves the device.
- **Only extraction crosses the network boundary** — and only after the local PHI redaction step has completed.
- **Audio is also handled locally** — MacWhisper transcription and [[concepts/omlx-server]] summarization run on-device, eliminating a common PHI leakage vector.

This architecture means that even if the software-level redaction missed an identifier, the data would still be contained on the clinician's own machine rather than transmitted to a cloud provider. The Streamlit app's pipeline order is: Parse (local, PHI redacted) → Extract (Anthropic API, PHI-free) → Index (local SQLite + LanceDB) → Report (local rendering).

The [[concepts/luria-neuropsych-pipeline]] and the Streamlit app share a common structural principle: the local-first approach means the attack surface for PHI leakage is dramatically reduced even before software-level redaction is considered. Docling (see [[concepts/docling-pdf-parsing]]) serves as the local parse stage in both contexts, extracting text and layout from PDFs with PHI redacted before any content reaches a cloud API.

## The PDF Ingestion Entry Point

In pipelines such as the Luria neuropsychological system, the very first stage of document intake is the **PDF Ingestion & Parser Worker** — specifically, the `neuropsych-pdf-parser` agent (see [[summaries/neuropsych-pdf-parser]] and [[summaries/AGENTS_luria]]). This agent is responsible for fetching, classifying, and extracting raw PDF content before any downstream processing occurs. Critically, it is also the **first line of PHI defense**: the parser replaces identifying information with standardized tokens as part of its structured output, meaning PHI anonymization begins at the point of ingestion — before content reaches any reasoning agent.

In the Streamlit app context, [[concepts/docling-pdf-parsing]] performs an analogous role: it is the local parse stage that extracts text and layout from PDFs, with PHI redacted before any content is passed to Claude Sonnet for structured extraction.

### PHI Tokenization at Ingestion

The `neuropsych-pdf-parser` enforces a strict token substitution scheme:

| PHI Type | Replacement Token |
|---|---|
| Patient name | `[PATIENT]` |
| Date of birth | `[DOB]` |
| MRN, SSN, IDs | `[ID]` |
| Clinician/examiner name | `[CLINICIAN]` |
| Hospital/clinic name | `[FACILITY]` |
| Address, phone, email | `[CONTACT]` |

Year-only dates may be preserved when clinically necessary. Every substitution is logged in a `PHI_FLAGS` block appended to the parser's output, providing a complete audit trail of what was replaced and where. The parser also enforces a hard rule that it will **never** include real names, MRNs, DOBs, or addresses in its output, regardless of user instruction — making the PHI boundary a property of the agent itself, not just a downstream assumption.

The ingestion worker handles documents including neuropsychological assessment reports (WAIS-IV, MMSE, MoCA, RBANS, WMS, WCST, CPT, Trail Making, Stroop, BDI, BRIEF), psychometric score sheets, clinical notes, and research papers — all of which may carry PHI in different formats and locations within the document. See [[concepts/neuropsychological-tests]] for context on these instruments.

## Implementation in the Luria Pipeline

The [[summaries/deepagents_merged_mem_notes]] describes a concrete implementation of this pattern in the Luria neuropsychological report system:

### Tools Used

- **`neuropsych-pdf-parser`** (Stage 1) — the entry-point agent that performs inline PHI tokenization during document ingestion, producing a structured, PHI-free text block before any LLM reasoning occurs (see [[summaries/neuropsych-pdf-parser]])
- **Microsoft Presidio** (`tools/pii_presidio.py`) — the primary redaction engine for transcript-derived content. Detects and masks:
  - Patient names
  - Date of birth
  - Addresses
  - Phone numbers
  - Medical Record Numbers (MRN)
  - Social Security Numbers (SSN)
- **Docling** — used for initial document parsing (PDF extraction) before redaction is applied (see [[concepts/docling-pdf-parsing]])

### Where the Gate Lives: Phase A5

In the Luria pipeline's Phase A (NSE Intake), the redaction gate is placed at node **A5** (`nse_cod_summary`):

- **A4** produces `data/intake/nse_transcript.md` — the raw STT transcript containing full names, DOB, and other identifiers captured during the clinical interview
- **A5** runs Presidio over this transcript and outputs `data/intake/nse_summary_redacted.md`
- **All subsequent agents** (A6 through Phase D) receive only the redacted output

### Filesystem Permission Model

Complementing the software redaction, the pipeline enforces a HIPAA-aware filesystem permission model:
- `data-raw/` is **denied** to all subagents
- Only human-supervised or explicitly authorized agents can access raw PHI directories
- This defense-in-depth approach means even if redaction missed an identifier, subagents cannot reach the raw source files

The Streamlit app implements a simpler but analogous model: raw uploaded PDFs land in `data/uploads/` and processed outputs go to `data/reports/`, `data/neuropsych.db`, and `data/vectors/` — all gitignored and local.

### Quality Review Re-scan

At Phase D2 (`luria-quality-review`), Presidio is run a **second time** as a re-scan over the fully assembled report before it is finalized. The D2 quality checklist explicitly includes:
- No PHI in narrative text (Presidio re-scan)
- Validity statements present when required
- Score ↔ narrative consistency

## Key Design Decisions

### Redaction as a Pipeline Node, Not a Model Instruction

A common but unreliable approach is to instruct LLMs via prompt to avoid repeating PII. The Luria architecture correctly treats redaction as a **deterministic pipeline node** — a Python tool call or rule-based agent step with verifiable output — rather than relying on model behavior. This is essential for [[concepts/phi-data-handling]] compliance in clinical settings.

The `neuropsych-pdf-parser` exemplifies this principle: its PHI tokenization is implemented as a rule-based extraction step with explicit token substitution and audit logging, not a model-level instruction that might be ignored or forgotten. Its hard rules — never output real names regardless of user instruction, never round or paraphrase scores — are agent-level constraints, not prompt suggestions.

The Streamlit app's LangGraph pipeline (see [[concepts/langgraph-agent-workflows]]) similarly encodes PHI redaction as the first node in the `StateGraph`, ensuring the constraint is structural rather than advisory. This integration with [[concepts/agent-pipeline-state-management]] means the redacted content propagates through the state graph as a first-class artifact, not as an afterthought.

### Layered Anonymization

Both the full Luria pipeline and the Streamlit app employ anonymization at multiple layers:
1. **Ingestion layer** — the parse stage (whether `neuropsych-pdf-parser` or Docling in the Streamlit app) replaces PHI with tokens on first contact with the document
2. **Transcript gate (A5)** — Presidio sweeps speech-derived transcripts before LLM reasoning begins (full pipeline only)
3. **Filesystem layer** — raw data directories are access-controlled at the OS level
4. **Network boundary** — only PHI-free content crosses to cloud APIs
5. **Quality review (D2)** — a final Presidio re-scan before report delivery (full pipeline only)

### Audio-Derived Content: A Special Risk Vector

The [[concepts/audio-transcription-pipeline]] introduces a particularly high-risk PHI surface. The NSE transcript from Phase A4 (speech-to-text) captures full names, family member names, addresses, and dates of birth in natural conversational flow. In the Streamlit app, audio uploads are transcribed via MacWhisper CLI (local, on-device) and summarized via a local oMLX server — meaning the audio PHI surface never crosses a network boundary at all. This local-only handling of audio is consistent with the [[concepts/privacy-first-software]] design philosophy.

### Gap Identified

The corrected orchestration plan explicitly notes that `pii_presidio.py` **exists as a tool but was not called anywhere in the graph** as of the May 2026 review. Wiring it into A5 was listed as the second-highest build priority, after extending the LangGraph fan-out for domain parallelism. This illustrates a common failure mode: the redaction tooling exists but isn't integrated into the execution graph. The `neuropsych-pdf-parser`'s approach of making PHI scrubbing intrinsic to the agent's output contract — rather than a separate tool call — partially mitigates this risk at the ingestion layer.

## Relation to Broader Architecture

This pattern fits within the larger context of [[concepts/multi-agent-orchestration]], where subagents have narrow, isolated responsibilities and cannot be granted broad data access. The combination of:
1. PHI tokenization at the ingestion entry point
2. A hard redaction gate at a defined pipeline stage
3. Filesystem-level access denial to raw data directories
4. Local-only storage for vectors and structured data (see [[concepts/local-first-architecture]])
5. A downstream re-scan during quality review

...forms a layered [[concepts/privacy-first-software]] approach suited to HIPAA-regulated clinical environments.

The document classification step in the `neuropsych-pdf-parser` (assigning one of five document types: `neuropsych_assessment_report`, `psychometric_score_sheet`, `clinical_notes`, `research_paper`, `mixed_other`) also informs which PHI patterns are most likely present — clinical notes, for instance, embed PHI far more densely in running prose than research papers do.

## Related Pages

- [[summaries/neuropsych-pdf-parser]] — Stage 1 ingestion agent with hard PHI tokenization rules
- [[summaries/AGENTS_luria]] — PDF ingestion worker with inline PHI tokenization
- [[summaries/deepagents_merged_mem_notes]] — source implementation details
- [[summaries/README]] — Luria Streamlit App with local-first PHI redaction architecture
- [[concepts/phi-data-handling]] — broader PHI compliance considerations
- [[concepts/privacy-first-software]] — design philosophy
- [[concepts/multi-agent-orchestration]] — pipeline context
- [[concepts/neuropsychological-reporting]] — clinical domain context
- [[concepts/clinical-nlp-pipelines]] — NLP pipeline architecture
- [[concepts/neuropsychological-assessment-pipeline]] — the multi-stage pipeline this pattern operates within
- [[concepts/redaction-tokens]] — token substitution standards
- [[concepts/luria-neuropsych-pipeline]] — the specific pipeline implementation
- [[concepts/docling-pdf-parsing]] — local PDF parsing stage where redaction begins
- [[concepts/langgraph-agent-workflows]] — structural enforcement of redaction as a graph node
- [[concepts/local-llm-inference]] — on-device processing that reduces PHI exposure surface
- [[concepts/audio-transcription-pipeline]] — audio PHI surface and local-only handling
- [[concepts/agent-pipeline-state-management]] — how redacted state propagates through the graph
- [[concepts/lancedb-vector-store]] — local vector storage that keeps indexed data on-device
- [[concepts/omlx-server]] — local LLM inference used in audio summarization

See also: [[summaries/SKILL]]

See also: [[summaries/clinical-validity-reviewer]]

See also: [[summaries/neuropsych-data-extractor]]

See also: [[summaries/SESSION_SUMMARY_2025-04-28]]

See also: [[summaries/nse_narrative]]

See also: [[summaries/AUTISM_RAG_SYSTEM_DOCUMENTATION]]

See also: [[summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_CODEMAP]]

See also: [[summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_PIPELINE]]