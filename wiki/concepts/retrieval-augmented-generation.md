---
sources: [summaries/agentic-workflows.md, summaries/redesign_20260623110910.md, summaries/redesign_20260623110817.md, summaries/Introducing-FrontierCode.md, summaries/SESSION_SUMMARY.md, summaries/RECOMMENDATION_FILTERING_IMPROVEMENTS.md, summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_PIPELINE.md, summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_CODEMAP.md, summaries/AUTISM_RAG_SYSTEM_DOCUMENTATION.md, summaries/2026-02-11-this-session-is-being-continued-from-a-previous-co.md, summaries/SKILL.md, summaries/DEPENDENCIES.md, summaries/full-pipeline.md, summaries/customization.md, summaries/style-training-to-report-drafting.md, summaries/vector-store.md, summaries/text-extraction.md, summaries/style-trainer.md, summaries/soul-style-agent.md, summaries/report-generator.md, summaries/embedding-client.md, summaries/0009-soul-local-llm-inference-with-omlx.md, summaries/0008-soul-single-file-style-agent-architecture.md, summaries/0004-soul-style-profile-json.md, summaries/0002-soul-sqlite-vector-storage.md, summaries/0001-voice-record-architecture-decisions.md, summaries/conversation-export.md, summaries/WORKFLOW_INSTRUCTIONS.md, summaries/TECHNICAL_DOCS.md, summaries/SHINY_APP_FIXED.md, summaries/REBUILD_FINAL_STATUS.md, summaries/REBUILD_COMPLETE.md, summaries/README_WORKFLOW.md, summaries/README_PIPELINE.md, summaries/README_AS_PROCESSING.md, summaries/QUICK_REFERENCE.md, summaries/POSITRON_DATABOT_TROUBLESHOOTING.md, summaries/OCR_PDF_GUIDE.md, summaries/KNOWLEDGE_BASE_EXPLAINED.md, summaries/EMBEDDINGS_COMPLETE.md, summaries/COMPLETE_STATUS.md, summaries/AS_PROCESSING_COMPLETE.md, summaries/mlx_embeddings.md, summaries/index.md, summaries/mcp-integration.md, summaries/002-mcp-llm-integration.md, summaries/README.md, summaries/AGENTS_luria.md, summaries/README_luria.md, summaries/deepagents_merged_mem_notes.md, summaries/SESSION_SUMMARY_2025-04-28.md, summaries/PROJECT_SETUP_COMPLETE.md, summaries/DEMO_GUIDE.md]
brief: RAG pattern: retrieve relevant passages from a knowledge base to ground LLM outputs in verifiable source material.
---

# Retrieval-Augmented Generation (RAG)

Retrieval-Augmented Generation (RAG) is an architecture pattern that enhances large language model (LLM) outputs by first retrieving relevant passages from a knowledge base and injecting them into the prompt context before generation. This grounds the model's responses in specific, verifiable source material rather than relying solely on parametric (trained) knowledge.

## Core Mechanism

1. **Embed** documents into a vector store at ingest time
2. **Query** the vector store with a semantic similarity search at inference time
3. **Inject** the top-k retrieved chunks into the LLM prompt as context
4. **Generate** a response that is constrained to the provided context

This pattern is fundamental to [[concepts/clinical-nlp-pipelines]] and any system where factual accuracy, privacy, and source attribution matter.

---

## Shared Reference Materials and the assessments/neurocognitive Directory

The `assessments/neurocognitive` directory functions as a centralized repository of neurocognitive assessment materials — including interpretation guides, test manuals, and reference documents — shared across all patient RAG systems. Files placed here (PDF, DOCX, or plain text) are automatically ingested whenever any patient's RAG system is rebuilt, ensuring consistent access to clinical reference material without duplication. This directory is documented in [[summaries/README]] and represents a population-level reference layer analogous to the Luria KB described below.

---

## RAG in the Luria Neuropsych Ecosystem

The Luria system employs RAG across multiple tiers and subsystems. The **Luria KB** (`kb/`) is the knowledge layer that supplies curated clinical reference material to Luria's agents — diagnostic criteria (DSM-5-TR), score classifications (Wechsler, Heaton), validity testing guidance, HIPAA/APA compliance standards, ICD-10 coding, base rates, and RCI. This reference layer uses a distinctive approach: rather than embeddings-based vector search, the entire curated wiki (~250 KB, ~40 notes) fits in a single cached system prompt, enabling **long-context retrieval with prompt caching** via Claude. Citations are returned as exact quoted spans from source notes.

This KB-level retrieval is queried on every report — supplying approximately 10–15% of each draft that requires evidence grounding (diagnoses, recommendations, validity language). The long-context retrieval approach trades the chunking artifacts of embeddings-based RAG for wholistic, cross-note answers on a warm cache, making it faster and cheaper for a small, stable corpus queried constantly.

The broader system (the Luria Streamlit App and soul agent) uses conventional embeddings-based RAG for larger, heterogeneous corpora. The [[summaries/DEMO_GUIDE]] document describes two distinct uses of embeddings RAG within the Luria application. The [[summaries/PROJECT_SETUP_COMPLETE]] document reveals that the RAG module (`rag/`) is a first-class component of the Luria architecture, with an anticipated migration path into `src/luria/rag/` as the project matures. The [[summaries/deepagents_merged_mem_notes]] document adds a third use case: evidence-based recommendations retrieval at the SIRF phase.

The [[summaries/README]] document (the Luria Streamlit App README) provides the most complete architectural overview of how embeddings RAG is integrated into the full application. The app exposes RAG through two tabs — **Ask** (factual Q&A) and **Knowledge Base** (browsing indexed data) — underpinned by a 4-stage LangGraph ingest pipeline that populates the dual-store backend.

### Why Two Retrieval Methods?

The KB architecture makes an explicit right-sizing argument:

- **Long-context + caching** (wiki tier): small, hand-curated, queried constantly — no chunking artifacts, wholistic cross-note answers, warm cache hits
- **Embeddings + vector search** (store tier): large, heterogeneous sub-corpora queried sporadically — embeddings cost is justified only where data volume demands it

This two-method design is captured in the [[concepts/knowledge-base-architecture]] and [[concepts/local-first-architecture]] patterns that shape the broader Luria system.

---

## RAG in the Luria App Redesign: Clinical Office and Console

The [[summaries/redesign_20260623110817]] and [[summaries/redesign_20260623110910]] redesign documents introduce RAG-powered interfaces that extend the pattern into the clinical workspace across eight functional modules.

### Clinical Office (Document Ingestion + RAG)

The **Clinical Office** (Page 04 of the redesign) is the document indexing hub. Records are ingested via **markitdown** (PDF/file → Markdown conversion), then processed through a RAG pipeline with `RAG · 24 chunks embedded`. Each indexed source document yields extracted facts linked to their source page and page number:

- *"Bright but can't keep his pencil on the line."* ↳ `Clinical Interview`
- Expressive speech delay treated age 3–4. ↳ `Medical Records · p.3`
- Paternal history of ADHD. ↳ `Clinical Interview`

These grounded facts pre-fill the report background section automatically, eliminating manual transcription. Documents carry status indicators: `INDEXED`, `PROCESSING`, `NOT ON FILE`. The ingestion pipeline uses markitdown for conversion, aligning with the OpenKB short-document ingestion approach described in Page 10 of the redesign.

Document types supported: Clinical Interview transcripts, Prior Medical Records, School Records/IEP, and Prior Testing. The RAG pipeline extracts discrete clinical facts from each source (e.g., "1 transcript · 6 facts extracted"; "8 pages · 4 facts extracted") and flags missing records (e.g., no prior psychoeducational evaluation) for follow-up.

### Console & Synthesis (Grounded Clinical Reasoning)

The **Console** (Page 05) is described as "a clinical copilot you argue with." It implements RAG-grounded conversational reasoning: every Luria response cites numbered sources from an **Evidence Rail** visible alongside the conversation. Example:

> *"Is the inattention primary ADHD, or secondary to the dysgraphia and academic frustration?"*
>
> Luria answers with three lines of evidence, each numbered: Conners-4 cross-setting ratings¹, NEPSY-II dissociation from graphomotor output², pending Beery VMI³, and caregiver interview onset history⁴.

This citation-to-source mapping is the defining feature of the Console: RAG grounding is surfaced visibly to the clinician, not hidden in a system prompt. Actions include `[Add to Synthesis]`, `[Draft this section]`, and `[Show reasoning]`. The Console operates with context across 6 tests, 4 docs, and live terminal access.

The Console's evidence rail pattern represents a mature application of [[concepts/clinical-ai-reasoning]] — grounded, auditable, and contestable by the clinician.

### Intake Assistant (Structured History RAG)

The **Patient Intake** module (Page 02) features an AI intake assistant that auto-structures referral text into discrete concern tags (`Inattention`, `Graphomotor / handwriting`, `Academic underperformance`, `r/o ADHD`, `r/o Dysgraphia`) and flags missing data gaps. This lightweight retrieval-and-reasoning layer pre-populates structured fields from unstructured clinical notes, connecting to [[concepts/neurodevelopmental-clinical-intake]] workflows.

### Cognitive Map (Pattern Detection)

The **Cognitive Map** (Page 07, Amber theme) visualizes 13 neurocognitive domains as a constellation and applies AI pattern detection across the full score profile. For the sample case, Luria identifies a "Frontal-graphomotor cluster" — attention, executive control, and motor output co-varying and jointly depressed while verbal reasoning is spared — and names this the classic ADHD-Inattentive + dysgraphia signature. This pattern synthesis draws on the same RAG-grounded evidence base as the Console.

### Report Builder (Citation-Grounded Drafting)

The **Report Builder** (Page 08) drafts clinical report sections from 6 tests and 3 sources with citations enabled. Controls include voice (Clinical / Balanced / Parent), reading level (Grade 8 to Professional), and section inserts (score tables, domain figures, DSM-5 criteria). The live synthesis panel on the Report Workspace generates provisional impressions in real time: "Two findings converge on the referral question. Attention/Executive (SS 76) and graphomotor output are both depressed, while verbal reasoning is intact — a profile consistent with ADHD-Inattentive with co-occurring dysgraphia rather than a global delay."

### OpenKB as Proposed KB Backend

Page 10 of the redesign documents a proposed (not yet implemented) integration of **OpenKB** as the knowledge backend for Luria. OpenKB is described as an open-source system that compiles raw documents into a persistent, interlinked wiki using LLMs, powered by **PageIndex** for vectorless long-document retrieval.

The key architectural distinction from traditional RAG:
- **Traditional RAG**: rediscovers knowledge from scratch on every query; nothing accumulates
- **OpenKB**: compiles knowledge once into a persistent wiki, then keeps it current; cross-references pre-exist, contradictions are flagged, synthesis reflects everything consumed

OpenKB has two layers: a **wiki foundation** (compile and maintain knowledge) and **generators** (query / chat / Skill Factory). For long PDFs (≥20 pages), PageIndex builds a hierarchical tree index with summaries; the LLM reads the tree instead of full text. Short documents are read in full after markitdown conversion.

Knowledge compilation steps when a document is added:
1. Generate a **summary** page
2. Read existing **concept** and **entity** pages
3. Create or update concepts with cross-document synthesis
4. Create or update **entity** pages (people, orgs, places, products)
5. Update the **index** and **log**

Generators include: `openkb query` (grounded single-question answers), `openkb chat` (multi-turn session with history), and `openkb skill new` (distills wiki into an Anthropic Skill portable folder for Claude Code, Codex CLI, Gemini CLI, and Cursor). The Skill Factory produces redistributable skill folders with structural validation, trigger-accuracy evaluation, and full history/rollback quality gates.

This OpenKB integration would position Luria's KB layer as a continuously compiled, self-updating clinical knowledge graph — rather than a static prompt cache — directly addressing the knowledge accumulation gap in traditional RAG. See [[concepts/knowledge-base-architecture]] and [[concepts/luria-skills]] for related patterns.

---

## RAG in the Luria Redesign: LLM Infrastructure

The redesign's Page 09 documents the full LLM infrastructure underpinning RAG in the new Luria app. The `LocalFallbackLLMClient` implements a **provider fallback chain** for all LLM calls (including RAG generation):

1. **oMLX** — local OpenAI-compatible API (PHI-safe)
2. **vMLX** — local Responses API (PHI-safe)
3. **Ollama** — local native API (PHI-safe)
4. **Cloud** — remote fallback, non-PHI only (blocked by `restrictToPreferredProviders: true`)

A critical PHI guard — `redactPhi()` — sits upstream of any LLM call in the `/stt/summarize` route, ensuring patient data is de-identified before generation. `llmAbortContext` (AsyncLocalStorage) threads an abort signal through `generate()` so a RAG job can be cancelled mid-fallback.

All patient data stays on-device through the first three providers; cloud is a PHI-blocked last resort. This architecture directly embodies [[concepts/local-first-architecture]] and [[concepts/phi-data-handling]] requirements.

The data flow for RAG-powered report generation:
1. `IntakeDossier.tsx` commits dossier to encrypted SQLite
2. `redactPhi()` guards the LLM boundary
3. `LocalFallbackLLMClient.generate()` with `restrictToPreferredProviders: true` keeps inference local
4. `agentRunner.ts` dispatches section agents (`nseCodSummary`, `ROCFT`, report-section agents)
5. Each agent invokes RAG retrieval then generation through the fallback chain

See [[concepts/luria-neuropsych-pipeline]] and [[concepts/luria-overview]] for the broader system context.

---

## RAG in the Neuropsychological Report Analysis Pipeline

The neuropsychological report analysis pipeline (documented in [[summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_CODEMAP]]) implements a full six-stage RAG pipeline purpose-built for clinical recommendation generation from de-identified neuropsychological reports.

### Pipeline Architecture Overview

1. **PDF Ingestion** — PyMuPDF extracts raw text from clinical PDF reports
2. **Report Parsing** — `report_parser.py` extracts metadata, diagnoses (DSM-5 with ICD-10 mapping), and recommendation sections
3. **PHI De-identification** — `report_deidentify.py` removes patient names, dates, and case numbers through layered regex replacement
4. **Recommendation Chunking** — Sections are split into `RecommendationChunk` objects by subsection header with attached metadata
5. **Embedding & Indexing** — SentenceTransformer encodes chunks; FAISS IndexFlatIP stores normalized vectors for cosine similarity search
6. **Search & Generation** — User queries are embedded, filtered metadata search retrieves candidates, and an LLM generates tailored recommendations

### DSM-5 Diagnosis Extraction

The `extract_diagnoses()` function handles **six code format patterns**:

- **Format 4 (name-first)**: `ADHD, Combined Type 314.01 (F90.2)` — name precedes codes
- **Format 1 (codes-first)**: `314.01 (F90.2) ADHD, Combined Presentation` — codes precede name
- **Format 2 (ICD-10 only)**: `F90.2 ADHD` — broadest pattern, tried last

After matching, `_merge_equivalent_diagnoses()` applies normalization via `canonicalize_diagnosis_name()` and `classify_dsm5_category()`. See [[concepts/dsm5-diagnosis-normalization]] and [[concepts/icd10-diagnosis-extraction]].

### PHI De-identification Pipeline

The `deidentify_recommendations()` function applies a multi-stage de-identification workflow ending in deterministic 6-character hash placeholders (`_short_hash()`), ensuring consistent anonymization across multiple reports for the same patient. See [[concepts/phi-deidentification-pipeline]] and [[concepts/pii-redaction-pipelines]].

### Recommendation Section Parsing

The `split_recommendations_by_subsection()` function splits content by subsection headers using three heuristics: PHASE X pattern, ALL CAPS pattern, and Title Case pattern. Each `RecommendationChunk` carries text, sub-header, diagnoses, age group, and context metadata. See [[concepts/rag-chunking]] and [[concepts/text-chunking]].

### Filtered Semantic Search

The `VectorStore.search_filtered()` function implements an over-fetch strategy (k×3 from FAISS, then metadata filtering). See [[concepts/faiss-vector-index]] and [[concepts/vector-search]].

### LLM-Powered Recommendation Generation

The `generate_recommendations()` function supports multiple providers via dynamic `importlib` loading: Anthropic Claude, OpenAI GPT, Ollama (local). See [[concepts/llm-provider-abstraction]] and [[concepts/clinical-narrative-generation]].

---

## RAG in the Autism-RAG System

The `autism-rag` project is a dual-pipeline RAG application designed for: (1) research document Q&A powered by FLAN-T5, and (2) clinical report parsing with LLM-powered recommendation generation. Both pipelines share a common retrieval infrastructure built on FAISS. See [[summaries/AUTISM_RAG_SYSTEM_DOCUMENTATION]].

### Pipeline 1: Research Document Q&A

- Chunks text into **800-word segments with 120-word overlap** via [[concepts/rag-chunking]]
- Generates **384-dimensional embeddings** using `all-MiniLM-L6-v2` (SentenceTransformers)
- Stores vectors in **FAISS IndexFlatIP** (see [[concepts/faiss-vector-index]])
- Generation uses **FLAN-T5** (`google/flan-t5-base`)

### Pipeline 2: Clinical Recommendation System

- PHI de-identification (see [[concepts/phi-deidentification-pipeline]]) before storage
- Supports multiple providers via [[concepts/llm-provider-abstraction]]: Anthropic Claude, OpenAI GPT, Ollama
- Connects to [[concepts/clinical-narrative-generation]] for tailored recommendation output

### Autism-RAG: Knowledge Base Curation

A critical curation decision: the textbook `essentials_report_writing_1sted_recs_only.pdf` was moved from the retrieval index into the LLM system prompt (`data/recommendation_guidelines.md`). Its dual presence caused polluted search results and circular generation. After re-ingestion the KB contained **104 reports → 426 chunks** of genuine clinical recommendations only.

**Key principle: system-prompt knowledge and retrieval-corpus knowledge must not overlap.**

### Autism-RAG: DSM-5 Normalization

A session of improvements ([[summaries/2026-02-11-this-session-is-being-continued-from-a-previous-co]]) fixed multiple canonicalization failures. Key insight: **ICD-10 code-based classification must precede name-based classification** — when a diagnosis name is garbled by PDF parsing, the numeric code is often still intact. Result: 72 → 66 unique disorders, all correctly categorized.

---

## RAG in the Luria Streamlit App

### 1. Clinical Data Query ("Ask" Tab)

After PDF ingestion, the pipeline indexes data into **LanceDB** (vector store) and **SQLite** (structured storage). When a clinician asks a question, the system performs semantic search over LanceDB, retrieving only chunks from the ingested patient report, preventing hallucination.

#### Indexing Architecture

- **SQLite (TestScores + ClinicalSummaries tables)** — structured psychometric data with exact scores
- **LanceDB vector store** — narrative text chunked by section headers, embedded using `sentence-transformers/all-MiniLM-L6-v2` (384-dimensional vectors)
- **Immutability rule** — existing rows are never overwritten; only append new data

### 2. Voice/Style Matching ("SOUL" Feature)

The "Luria Voice" feature applies RAG for **stylistic retrieval** rather than factual retrieval, implemented as a three-stage CLI pipeline:

```
PDF/TXT/MD reports ──► build-index ──► SQLite vector store
                                              │
                                      train-style ──► JSON style profile
                                              │
                              write-report ◄──┘
```

The soul profile is persisted as a **JSON file** (formalized in ADR 0004, [[summaries/0004-soul-style-profile-json]]). Its schema captures six fields: `voice`, `tone`, `structure_patterns`, `typical_phrases`, `do_rules`, and `avoid_rules`. See [[summaries/soul-style-agent]] and [[summaries/style-training-to-report-drafting]].

### 3. Evidence-Based Recommendations (Phase C3)

The `sirf_recs` agent (C3) queries both a PageIndex and a local `rag_db/` to retrieve evidence-based recommendations during SIRF report generation — grounding the recommendations section in published clinical literature rather than parametric knowledge alone.

### LangGraph Pipeline Architecture

Key entry points in `neuropsych_agent/graph.py`:

| Function | Purpose |
|----------|----------|
| `build_ingest_graph()` | 4-stage pipeline: parse → extract → index → report |
| `build_rag_graph()` | Single-node retrieval for Q&A |
| `ingest_pdf(path, mode="ingest", **voice_kw)` | Convenience wrapper used by the UI |
| `ask_rag(query)` | Convenience wrapper for RAG Q&A |

---

## Customizing RAG Pipelines

### Per-Clinician and Per-Population Style Profiles

- **Per-clinician profiles** trained exclusively from one clinician's historical reports
- **Per-population profiles** separate pediatric, adult, and forensic report corpora

### Chunking Strategy Customization

| Use Case | `--chunk-size` | `--overlap` |
|---|---|---|
| Long-form comprehensive reports | 2000 | 200 |
| Brief screening reports | 600 | 75 |
| Default (soul agent) | 1200 | 150 |
| Research Q&A (autism-rag) | 800 words | 120 words |

### Retrieval Parameter Tuning

| Mode | `--top-k` | `--temperature` |
|---|---|---|
| High-context (complex cases) | 12 | 0.15 |
| Quick drafting (standard cases) | 4 | 0.25 |
| Balanced (default) | 6 | 0.2 |

---

## The Style-Trainer: RAG for Systematic Style Extraction

The `train-style` command uses a **fixed semantic seed query** for consistent exemplar retrieval, assembles a corpus of 12 exemplar chunks with source attribution, and invokes the LLM at **temperature 0.1** for reproducible style extraction. See [[summaries/style-trainer]].

---

## RAG for PAI Psychological Assessment

The PAI RAG System is a purpose-built R-based application applying RAG to the Personality Assessment Inventory. It uses **Parquet files** as the persistent storage layer (DuckDB used only for runtime ad-hoc queries). After a January 29, 2026 rebuild the knowledge base grew from 2,546 to **4,830 chunks** from 98 documents.

**Architecture layers:**
- **Layer 1 — Retrieval**: Dual-path hybrid scoring: `hybrid_score = (semantic_score_norm × semantic_weight) + (text_score_norm × text_weight)`
- **Layer 2 — Context Formatting**: Top-K chunks with source and relevance scores
- **Layer 3 — LLM Integration**: Routed to Ollama (local), OpenAI, or Anthropic Claude via [[concepts/llm-provider-abstraction]]

**Embedding model:** `snowflake-arctic-embed2:568m` — 1,024 dimensions, locally served via Ollama.

See [[concepts/parquet-as-knowledge-store]], [[concepts/hybrid-search-retrieval]], [[concepts/pai-knowledge-base]], and [[concepts/pai-assessment]].

---

## Minimal-Stack RAG: SQLite-Only Architecture

The soul agent pipeline demonstrates a RAG architecture that avoids a dedicated vector database entirely:

- Embeddings stored as serialized JSON float arrays in a SQLite `chunks` table
- Cosine similarity computed in pure Python in-memory at query time
- Chunking: 1200 characters with 150-character overlap

This approach aligns with [[concepts/sqlite-as-vector-store]] for small-to-medium corpora. See [[summaries/0001‑choose‑local‑llm]], [[summaries/0002-soul-sqlite-vector-storage]], and [[concepts/single-file-agent-pattern]].

---

## RAG Pipeline Stages (Three-Stage Model)

| Stage | Command | Input | Output |
|-------|---------|-------|--------|
| 1. Build Index | `build-index` | Directory of PDFs/TXT/MD | SQLite DB with chunks & embeddings |
| 2. Train Style Profile | `train-style` | SQLite DB + seed prompt | `report_style_profile.json` (JSON) |
| 3. Generate Report | `write-report` | SQLite DB + profile + user prompt | Draft report text |
| 4. Render | `quarto render` | `.qmd` sections with pasted draft | PDF |

The `write-report` command embeds four safety guardrails: no fabrication, missing data marking (`[NEEDS DATA]`), style adherence, and clinician review framing.

---

## RAG vs. Fine-Tuning

RAG is generally preferred over fine-tuning when:
- The knowledge base changes frequently (new patient reports, new reference documents)
- Source attribution is required (clinical accountability)
- Privacy constraints prevent sending data to external model trainers
- Deployment needs to remain lightweight and updatable

In the Luria redesign, the Console's evidence rail makes RAG grounding explicit and auditable — a key differentiator from black-box generation.

---

## RAG and Population vs. Individual Data

Two dataset purposes must be kept separate:

- **General population reports** — ingested broadly to capture a wide range of report headers; makes the RAG module generalizable
- **Individual clinician reports** — used to weight the model and give the clinician their own voice; must **not** be blended with population data

This architectural boundary (generalizability vs. personalization as independent retrieval indexes) applies across Luria, the PAI system, and the autism-rag corpus hygiene principle.

---

## Key Components in a RAG System

| Component | Luria KB | Luria App | Soul Agent | PAI | autism-rag / NP Report | Luria Redesign |
|-----------|----------|-----------|------------|-----|------------------------|----------------|
| Document ingester | Hand-authored wiki notes | `index_node` in LangGraph | `build-index` command | R ingest scripts (ragnar) | `ingest_recommendations.py` / `ingest.py` | markitdown → Clinical Office |
| Retrieval method | Long-context + prompt caching | LanceDB vector search | SQLite cosine similarity | Hybrid semantic + keyword | FAISS semantic similarity | RAG · 24 chunks embedded |
| Vector/knowledge store | Cached system prompt | LanceDB | SQLite chunks table | Parquet/DuckDB (snowflake-arctic) | FAISS IndexFlatIP | Embedded in Clinical Office |
| Structured store | — | SQLite (TestScores, ClinicalSummaries) | Soul profile JSON | AS_scores_template.json | KB JSON metadata + RecommendationChunk | Encrypted SQLite |
| Generator | Claude (Opus) | Claude API / local oMLX | OMLX local / Ollama fallback | LLM interpretation layer | FLAN-T5 / Claude/GPT/Ollama | LocalFallbackLLMClient (oMLX/vMLX/Ollama/Cloud) |
| PHI handling | — | PHI redacted before Claude call | Style-only, no patient facts | Local Ollama only | Deterministic hash de-identification | `redactPhi()` + `restrictToPreferredProviders: true` |

---

## Related Concepts

- [[concepts/clinical-nlp-pipelines]] — RAG is a core pattern in clinical NLP for grounded extraction and reporting
- [[concepts/knowledge-base-architecture]] — The two-tier KB design shapes when and how RAG is applied
- [[concepts/local-first-architecture]] — The soul agent's stdlib-only, on-device design exemplifies local-first RAG
- [[concepts/persistent-memory]] — Vector stores like LanceDB serve as the persistent memory layer that RAG queries
- [[concepts/privacy-first-software]] — Local RAG over local vector stores avoids sending sensitive data to external APIs
- [[concepts/pii-redaction-pipelines]] — PII redaction gates protect data before it enters RAG indexes
- [[concepts/phi-deidentification-pipeline]] — Embedded de-identification in the ingestion step before storage
- [[concepts/multi-agent-orchestration]] — RAG is a key tool capability invoked by subagents during domain interpretation
- [[concepts/transformer-architecture]] — Embedding models and LLMs used in RAG are transformer-based
- [[concepts/r-python-integration]] — Retrieved context from RAG can feed into R-based analysis and reporting layers
- [[concepts/neuropsychological-reporting]] — The primary applied domain for Luria's RAG implementations
- [[concepts/phi-data-handling]] — PHI constraints shape the local-first architecture of RAG storage layers
- [[concepts/sqlite-as-vector-store]] — A minimal-stack alternative to dedicated vector DBs for small corpora
- [[concepts/lancedb-vector-store]] — The vector store used for narrative chunk retrieval in the Luria Streamlit App
- [[concepts/faiss-vector-index]] — The vector store used in the autism-rag system and NP report pipeline
- [[concepts/style-profile-extraction]] — Soul training is a two-phase RAG workflow producing a reusable style artifact
- [[concepts/style-profiles]] — JSON profile artifacts that capture clinician voice and writing conventions
- [[concepts/local-llm-inference]] — RAG pairs with local inference to keep all data on-premises
- [[concepts/fallback-strategy]] — Ollama fallback ensures RAG pipelines remain operational when primary inference is unavailable
- [[concepts/pai-assessment]] — Psychological instrument whose interpretation is grounded via RAG retrieval
- [[concepts/pai-knowledge-base]] — The 98-document corpus powering PAI semantic search
- [[concepts/parquet-as-knowledge-store]] — The underlying storage format for PAI chunks and embeddings
- [[concepts/clinical-report-structure]] — RAG-retrieved contexts feed structured clinical report generation
- [[concepts/vector-search]] — The underlying similarity computation enabling semantic RAG queries
- [[concepts/hybrid-search-retrieval]] — The weighted combination of semantic and keyword search used in the PAI system
- [[concepts/ocr-pipeline]] — OCR limitations motivate manual T-score entry in the PAI interpretation pipeline
- [[concepts/llm-provider-abstraction]] — Multiple systems route to Ollama, OpenAI, or Anthropic via a unified interface
- [[concepts/duckdb-as-vector-store]] — DuckDB serves as a runtime query engine over Parquet-stored embeddings
- [[concepts/clinical-data-privacy]] — Privacy requirements drive local-first RAG architecture choices
- [[concepts/subagent-architecture]] — RAG retrieval nodes are implemented as subagents within the LangGraph orchestration
- [[concepts/langgraph-agent-workflows]] — The LangGraph StateGraph wires RAG as a discrete node in both ingest and Q&A pipelines
- [[concepts/omlx-server]] — The local inference server used for embedding and generation in the soul agent and Luria redesign
- [[concepts/openai-compatible-api]] — The soul agent uses an OpenAI-compatible API for OMLX embedding and generation calls
- [[concepts/single-file-agent-pattern]] — The soul agent's single-file design concentrates all RAG logic in one Python file
- [[concepts/rag-chunking]] — Chunk size and overlap parameters that govern how documents are split for indexing
- [[concepts/modular-report-architecture]] — Section-by-section generation maps to the modular report assembly pattern
- [[concepts/quarto]] — The rendering system that converts RAG-generated draft prose into polished PDF reports
- [[concepts/dsm5-diagnosis-normalization]] — Canonicalization of diagnosis names in the KB improves retrieval precision
- [[concepts/icd10-diagnosis-extraction]] — ICD-10 code extraction is a key step in clinical report RAG pipelines
- [[concepts/clipboard-api-patterns]] — Client-side clipboard integration for exporting RAG-generated recommendations
- [[concepts/clinical-narrative-generation]] — LLM-powered recommendation generation grounded in retrieved clinical examples
- [[concepts/age-group-classification]] — Metadata field used for filtered search in the autism-rag and NP report systems
- [[concepts/recommendation-rag-pipeline]] — The specialized RAG pipeline for neuropsychological recommendation generation
- [[concepts/text-chunking]] — Text splitting strategies underlying all RAG indexing workflows
- [[concepts/autism-research-rag]] — The autism-focused research Q&A and clinical recommendation generation system
- [[concepts/clinical-ai-reasoning]] — The Console's evidence rail surfaces RAG grounding as auditable clinical reasoning
- [[concepts/luria-overview]] — Overview of the Luria platform whose RAG layers this page documents
- [[concepts/luria-neuropsych-pipeline]] — The broader neuropsych pipeline within which RAG operates
- [[concepts/luria-skills]] — Skill Factory generator distilling the compiled wiki into portable Anthropic Skills
- [[concepts/knowledge-capture]] — OpenKB's compiled wiki approach to persistent, accumulating knowledge
- [[concepts/neurodevelopmental-clinical-intake]] — Structured intake workflow feeding RAG-grounded report generation
- [[concepts/clinical-ai-copilot]] — The Console interface embodying RAG-grounded conversational clinical reasoning
- [[summaries/redesign_20260623110817]] — Luria app redesign introducing the Clinical Office RAG interface and Console evidence rail
- [[summaries/redesign_20260623110910]] — Luria app redesign detailing the full module set and OpenKB KB backend proposal
- [[summaries/README]] — Autism-RAG README introducing the dual-pipeline system architecture
- [[summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_CODEMAP]] — Detailed six-stage codemap of the NP report analysis and recommendation RAG pipeline
- [[summaries/AUTISM_RAG_SYSTEM_DOCUMENTATION]] — Full documentation of the autism-rag dual-pipeline system
- [[summaries/2026-02-11-this-session-is-being-continued-from-a-previous-co]] — Session documenting clipboard feature, corpus curation, and DSM-5 normalization fixes in autism-rag
- [[summaries/customization]] — Comprehensive guide to per-clinician profiles, chunking tuning, retrieval parameters, and output formats
- [[summaries/style-training-to-report-drafting]] — Documents the full end-to-end pipeline from style training through Quarto PDF rendering
- [[summaries/style-trainer]] — Documents the `train-style` command: fixed seed query, 12-exemplar retrieval, temperature-0.1 JSON profile generation
- [[summaries/soul-style-agent]] — Documents the standalone style agent: three-stage CLI pipeline with SQLite RAG
- [[summaries/0004-soul-style-profile-json]] — ADR formalizing JSON as the storage format for RAG-derived style profiles
- [[summaries/0008-soul-single-file-style-agent-architecture]] — ADR documenting the single-file architecture decision for the soul agent
- [[summaries/0009-soul-local-llm-inference-with-omlx]] — ADR documenting OMLX as the inference backend for the soul agent
- [[summaries/0002-soul-sqlite-vector-storage]] — ADR documenting SQLite as the vector storage backend
- [[summaries/conversation-export]] — Hands-on build session for the original PAI RAG system
- [[summaries/TECHNICAL_DOCS]] — Detailed technical specification of the PAI RAG system
- [[summaries/REBUILD_FINAL_STATUS]] — Records the January 29, 2026 corpus rebuild confirming the Parquet-first architecture
- [[summaries/README_PIPELINE]] — Describes the automated PAI interpretation pipeline
- [[summaries/KNOWLEDGE_BASE_EXPLAINED]] — Clarifies that PAI storage uses Parquet files, not persistent DuckDB
- [[summaries/EMBEDDINGS_COMPLETE]] — Records completion of PAI embedding generation
- [[summaries/AGENTS_luria]] — Specifies the Sheets Data Indexer Worker that populates the SQLite + LanceDB RAG index
- [[summaries/DEMO_GUIDE]] — Demonstrates two RAG use cases: factual query and style matching
- [[summaries/PROJECT_SETUP_COMPLETE]] — Establishes `rag/` as a first-class Luria module
- [[summaries/deepagents_merged_mem_notes]] — Adds Phase C3 recommendations RAG and the population vs. individual corpus distinction
- [[summaries/0001‑choose‑local‑llm]] — ADR rationale for local LLM inference underpinning the soul agent's RAG backend
- [[summaries/0001-voice-record-architecture-decisions]] — Voice ADR log for architectural decisions including RAG infrastructure
- [[summaries/embedding-client]] — Embedding client implementation used in RAG pipelines
- [[summaries/mlx_embeddings]] — MLX-based embedding generation relevant to local RAG inference
- [[summaries/SESSION_SUMMARY_2025-04-28]]
- [[summaries/README_luria]]
- [[summaries/002-mcp-llm-integration]]
- [[summaries/mcp-integration]]
- [[summaries/OCR_PDF_GUIDE]]
- [[summaries/README_WORKFLOW]]
- [[summaries/REBUILD_COMPLETE]]
- [[summaries/WORKFLOW_INSTRUCTIONS]]
- [[summaries/text-extraction]]
- [[summaries/vector-store]]
- [[summaries/full-pipeline]]
- [[summaries/DEPENDENCIES]]
- [[summaries/SKILL]]

See also: [[summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_PIPELINE]]

See also: [[summaries/RECOMMENDATION_FILTERING_IMPROVEMENTS]]

See also: [[summaries/SESSION_SUMMARY]]

See also: [[summaries/Introducing-FrontierCode]]

See also: [[summaries/agentic-workflows]]