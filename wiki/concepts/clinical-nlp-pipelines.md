---
sources: [summaries/clinical-assessment.md, summaries/RECOMMENDATION_FILTERING_IMPROVEMENTS.md, summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_PIPELINE.md, summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_CODEMAP.md, summaries/DIAGNOSIS_PARSER_IMPROVEMENTS.md, summaries/DIAGNOSIS_FIX_SUMMARY.md, summaries/AUTISM_RAG_SYSTEM_DOCUMENTATION.md, summaries/AGE_OVERRIDE_GUIDE.md, summaries/2026-02-11-this-session-is-being-continued-from-a-previous-co.md, summaries/nse_narrative.md, summaries/DEPENDENCIES.md, summaries/text-extraction.md, summaries/README.md, summaries/neuropsych-narrative-writer.md, summaries/neuropsych-data-extractor.md, summaries/conversation-export.md, summaries/local_models.md, summaries/index.md, summaries/report-generation.md, summaries/mcp-integration.md, summaries/002-mcp-llm-integration.md, summaries/AGENTS_luria.md, summaries/README_luria.md, summaries/SESSION_SUMMARY_2025-04-28.md, summaries/RECOVERY_NOTES.md, summaries/DEMO_GUIDE.md]
brief: Clinical NLP pipelines turn clinical text into structured data and safe narrative output.
---

# Clinical NLP Pipelines

Clinical NLP pipelines are structured, multi-stage systems that process medical and clinical documents — transforming unstructured text such as neuropsychological PDF reports into structured data, retrieval-ready knowledge, and new clinical-quality narrative output. They are a core pattern in healthcare AI, where accuracy, privacy, auditability, and clinical interpretability are paramount. In practice, these pipelines support [[concepts/clinical-data-management]], [[concepts/neuropsychological-assessment-workflow]], and the broader demands of [[summaries/clinical-assessment]]: standardized evaluation, context-sensitive interpretation, and careful synthesis across multiple sources of evidence.

A key clinical point is that assessment data rarely speaks for itself. Especially in child and adolescent work, symptoms can vary across home, school, and clinic contexts, and informants often disagree. For that reason, strong clinical NLP pipelines should preserve source context, rater identity, and domain-specific structure rather than flattening everything into generic text blobs. This connects the engineering pipeline directly to [[concepts/multi-informant-assessment]] and [[concepts/cross-informant-correspondence]].

## Core Stages

Most clinical NLP pipelines follow a sequence of discrete processing nodes, each with a well-defined input and output:

### 1. Parse
Raw documents (PDFs, scanned files) are ingested and converted to plain text. This stage typically handles:
- Layout-aware extraction (tables, headers, figures)
- **PHI redaction** — removing Protected Health Information locally, so sensitive data never leaves the machine (see [[concepts/privacy-first-software]] and [[concepts/phi-data-handling]])
- Normalization of whitespace, encoding, and structure

In the Luria Streamlit App (see [[summaries/README]]) and [[summaries/DEMO_GUIDE]], this is handled by **Docling** — a local document parser with built-in PHI redaction (see [[concepts/docling-pdf-parsing]]). Critically, Docling runs entirely on-device; only after PHI redaction does any text leave the machine for cloud extraction.

The **PDF Ingestion & Parser Worker** defined in [[summaries/AGENTS_luria]] formalizes this stage as a dedicated agent with explicit responsibilities: fetching documents via URL or raw input, classifying the document type, extracting all meaningful text while preserving structure (sections, headings, tables, numerical values), and flagging and anonymizing PHI before any downstream processing. Crucially, this worker does **not** interpret or summarize — it only extracts and structures. All patient identifiers are replaced with `[PATIENT_ID]` tokens and clinician names with `[CLINICIAN]` tokens, enforcing a clean separation between ingestion and analysis (see [[concepts/pii-redaction-pipelines]]).

The **Neuropsychological Report RAG Codemap** (see [[summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_CODEMAP]]) and the **Neuropsychological Report RAG Pipeline** (see [[summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_PIPELINE]]) describe a parallel ingestion pipeline built on **PyMuPDF (`fitz`)** rather than Docling. Its `ingest_recommendations.py` orchestrates the full flow:

```text
PDF → fitz.open() → parse_report() → deidentify_recommendations() → generate_embeddings() → FAISS
```

The key design principle is identical: PHI is stripped from text chunks *before* they are embedded and stored in the vector index. Only de-identified recommendation chunks ever enter the vector store.

From a clinical-assessment perspective, the parse stage also has to preserve contextual distinctions that matter later: whether content came from history, testing, behavioral observations, parent report, teacher report, or final impressions. This is especially important when pipelines are later used to reason about [[concepts/cognitive-domains]], [[concepts/behavioral-rating-scales]], or attention concerns that may differ by setting.

#### Text Extraction Implementation

At the code level, plain-text extraction is handled by the `extract_text(path: Path) -> str` function in `soul/neuro_report_style_agent.py` (lines 54–67). It supports two file formats natively:

| Format | Extension | Implementation |
|--------|-----------|----------------|
| Plain Text | `.txt`, `.md` | Native Python `path.read_text()` |
| PDF | `.pdf` | PyPDF2 `PdfReader` |

Error handling is explicit and fail-loud:
- **Unsupported file type**: raises `ValueError` with `f"Unsupported file type: {path}"`
- **Missing PyPDF2**: raises `RuntimeError` with installation instructions
- **Encoding issues**: falls back to UTF-8 with `errors="ignore"`

File discovery across a reports directory is performed by the `iter_report_files()` generator, which uses `rglob` to recursively match `*.pdf`, `*.txt`, and `*.md` patterns.

### 2. Extract (Score Structuring)
Structured information is pulled from the normalized text using a language model or rule-based agent. In the Luria pipeline, this stage is handled by the **neuropsych-data-extractor** — a dedicated stage-2 agent that converts clean parsed text into [[concepts/long-format-clinical-data]] rows consumable by the cingulate engine via DuckDB's `read_csv_auto` loader.

In the Luria Streamlit App, **Claude Sonnet (Anthropic API)** performs this extraction, converting neuropsychological narrative into structured JSON records containing test scores and clinical summaries. The prompt used for this stage is defined in `subagents/Neuropsych_Data_Extractor/AGENTS.md` — a static file that must be present at a resolved workspace path for the pipeline to start (see [[summaries/RECOVERY_NOTES]] for details on path-resolution failures that can silently break this stage).

Clinical extraction involves:
- Named entity recognition for diagnoses, test scores, medications
- Converting narrative prose into structured, long-format CSV rows
- Mapping to standard clinical terminologies (T-scores, percentiles, DSM criteria)
- Preserving source-specific context that may support multi-informant interpretation

This stage should be thought of as supporting clinical assessment rather than replacing it. In standardized evaluation, the choice of measurement method matters: dimensional rating scales often produce stronger cross-informant correspondence than categorical classifications. That means extraction pipelines should preserve whether data came from dimensional scales, formal diagnoses, checklists, or narrative observations, because those distinctions affect downstream interpretation.

#### Diagnosis Extraction and Header Recognition

A recurring challenge in clinical report parsing is that diagnostic sections use highly variable header formats across institutions and clinicians. In neuropsychological reports, diagnostic information may appear under headers such as `DIAGNOSTIC CONSIDERATIONS`, `Diagnostic Summary`, `DSM-5 Diagnoses`, or `Axis I Diagnostic Considerations:` — as commonly found in reports using [[concepts/multiaxial-diagnosis-format]].

The `src/report_parser.py` module was enhanced to address a specific problem: 24 reports (23% of a clinical dataset) initially returned zero diagnoses despite containing valid diagnostic sections. The fixes applied were:

1. **New section header patterns**: Added recognition for `Diagnostic Considerations`, `Diagnostic Summary`, `DSM-5 Diagnoses`, and `DSM-IV Diagnoses`.
2. **Flexible ICD-10 code matching**: Extended regex patterns to accept both decimal (`F90.2`) and non-decimal (`F90`) ICD-10 formats (see [[concepts/icd10-diagnosis-extraction]]).
3. **Subsection header filtering**: Added logic to remove false positives — lines like `Axis I Diagnostic Considerations`, `Axis I Rule Out`, and `Axis II` — that would otherwise be captured as diagnosis entries.

The impact of these changes reduced reports with zero diagnoses from ~23% to ~9–11% of the dataset. Remaining zero-diagnosis reports are expected to be textbook examples, non-standard format reports, or recommendations-only documents.

The Neuropsychological Report RAG Pipeline extends this further with **six distinct code format patterns** handled by `extract_diagnoses()` in `report_parser.py`:

| Format | Example |
|---|---|
| Name-first (FMT4) | `ADHD, Combined Type 314.01 (F90.2)` |
| Codes-first (FMT1) | `314.01 (F90.2) ADHD, Combined Presentation` |
| ICD-10 only (FMT2) | `F90.2 ADHD` |

The broadest pattern (ICD-10 only) is tried last to reduce false positives. Extracted diagnoses are then passed through `_merge_equivalent_diagnoses()`, which calls `canonicalize_diagnosis_name()` and `classify_dsm5_category()` before deduplication — a pipeline of normalization, classification, and merging that produces a clean canonical diagnosis list. The `classify_dsm5_category()` function assigns diagnoses to one of **12 DSM-5 categories** using both ICD-10 code ranges and name-based fallback matching.

#### Diagnosis Name Canonicalization

A critical sub-problem in clinical extraction is **diagnosis normalization**: the same condition may appear under many names across reports (e.g., "Post-Traumatic Stress Disorder" vs "Posttraumatic Stress Disorder", "Specific spelling disorder" vs the DSM-5 canonical SLD name, or garbled parsing artifacts like "Expression, 315.2 (F81.81); and Mathematics"). The `canonicalize_diagnosis_name()` function in `src/report_parser.py` addresses this systematically. See [[concepts/dsm5-diagnosis-normalization]] for full coverage.

Key lessons from production use:
- **ICD-10 code-based classification must run before name-based classification**: when a diagnosis name is garbled by PDF parsing, the numeric code (DSM or ICD-10) is often still intact in the text. Adding code-first detection catches artifacts that name-based matching cannot handle (e.g., F81.81 → SLD Written Expression regardless of the surrounding label).
- **ICD-10 code ranges must be mapped precisely to DSM-5 chapters**: a single overly broad regex like `F9[0-9]` incorrectly lumped Conduct Disorder (F91), Tic Disorders (F95), and Enuresis (F98) under Neurodevelopmental Disorders. Correct mappings: F91 → Disruptive, Impulse-Control, and Conduct Disorders; F95 → Neurodevelopmental; F98 → Other/Unspecified.
- **String normalization creates merge/split hazards**: `_diagnosis_match_key()` normalization converts "Post-Traumatic" → "post traumatic" (two tokens) vs "Posttraumatic" → "posttraumatic" (one token). Any substring check that only tests one variant will miss the other. Always check both after normalization.

The `_dsm5_category_from_code()` function uses code ranges to assign DSM-5 chapter categories, with name-based classification as a fallback:

| ICD-10 Range | DSM-5 Category |
|---|---|
| F70–F79 | Intellectual Disability (Neurodevelopmental) |
| F80–F89 | SLD, ASD, Language/Speech (Neurodevelopmental) |
| F90, F95 | ADHD, Tic Disorders (Neurodevelopmental) |
| F91, F63 | Disruptive, Impulse-Control, and Conduct Disorders |
| F20–F29 (excl. F21) | Schizophrenia Spectrum and Other Psychotic Disorders |
| F32–F33, F34.8x | Depressive Disorders |
| F31, F34.0 | Bipolar Disorders |
| F40–F41 | Anxiety Disorders |
| F43 | Trauma- and Stressor-Related Disorders |
| F98 | Other / Unspecified (Elimination disorders) |

Note: F21 (Schizotypal) is deliberately excluded from the Schizophrenia Spectrum range and falls through to the name-based Personality Disorders check, reflecting DSM-5's cross-listing convention.

#### Knowledge Base Hygiene: Source Material Separation

A related pipeline concern is **source material contamination**. In the Autism RAG system (see [[summaries/AUTISM_RAG_SYSTEM_DOCUMENTATION]]), a clinical textbook PDF was initially ingested into the same vector store as real clinical reports. This caused two problems:
- Polluted search results — textbook examples surfaced alongside genuine clinical recommendations
- Circular generation — retrieved textbook snippets were fed back as "reference recommendations" to an LLM already guided by those same principles via the system prompt

The fix was to move the textbook out of the ingestion directory entirely, embedding its guidance only in the LLM system prompt, not in the retrieval index. This illustrates a general principle: **reference/guidance material belongs in the prompt layer; example/case material belongs in the retrieval layer**. See [[concepts/retrieval-augmented-generation]] and [[concepts/knowledge-base-architecture]] for broader treatment.

This separation is especially important in clinical assessment settings, where pipelines may mix manuals, checklists, recommendation libraries, and actual patient reports. Conflating them can blur the difference between standardized reference material and case-derived evidence.

#### Long-Format Schema (Cingulate Canonical)
The neuropsych-data-extractor enforces a strict **long-format** discipline: every test/subtest measurement occupies exactly one row, with score-type variants (`scaled_score`, `t_score`, `standard_score`, `z_score`, `base_rate`) stored in a `score_type` column rather than spread across multiple columns. Wide-format output is explicitly forbidden. See [[concepts/neuropsychological-test-scores]] for more on the score types captured.

Required columns include:
- `test`, `test_name`, `scale` — full and short identifiers, plus composite index name
- `raw_score`, `score`, `score_type`, `percentile`, `range` — numeric and qualitative score data
- `domain`, `subdomain`, `narrow` — cognitive classification taxonomy
- `pass` — PASS theory tag (`Planning`, `Attention`, `Simultaneous`, `Successive`); see [[concepts/pass-theory]]
- `verbal`, `timed`, `rater`, `age_group` — test modality and administration metadata
- `description`, `doc_id`, `date` — provenance fields

Required fields (enforced by `domain_processing_utils.R:991`): `domain`, `rater`, `age_group`, `test`.

The inclusion of `rater` is clinically important, not just technically convenient. Multi-informant assessment depends on preserving who reported what, in which context, and through which instrument. If a pipeline collapses parent, teacher, self, and examiner data into a single undifferentiated summary, it destroys exactly the discrepancy patterns that may be clinically informative.

#### Score-Type Mapping
- "scaled score" / "ss" / range 1–19 → `scaled_score`
- "T-score" / mean 50, SD 10 → `t_score`
- "standard score" / mean 100, SD 15 → `standard_score`
- "z-score" → `z_score`

#### Range Classification (Percentile Thresholds)
| Percentile | Label |
|---|---|
| ≥ 91st | Above Average |
| 75–90 | High Average |
| 25–74 | Average |
| 9–24 | Low Average |
| 2–8 | Below Average / Borderline |
| < 2 | Impaired / Exceptionally Low |

#### Multi-Rater Instruments
Behavioral rating scales (BRIEF, CBCL, BASC, BDEFS) yield one row **per rater per scale**, with the `rater` column taking values `self`, `parent`, `teacher`, or `examiner`. This ensures multi-informant data remains distinguishable in downstream analysis.

This is directly aligned with [[concepts/multi-informant-assessment]] and [[concepts/cross-informant-correspondence]]. In clinical work, disagreement across raters is often expected, and may reflect context-specific functioning rather than simple noise. Pipelines should therefore preserve divergence instead of averaging it away.

The extraction schema used across the Luria pipeline captures document-level metadata (`doc_id`, `doc_type`, `date`, `referral_reason`, `examiner`), per-test score fields, and clinical summary fields. Every test and subtest is extracted as a **separate row** with exact numerical values preserved. Supported assessment instruments include WAIS-IV, MMSE, MoCA, RBANS, WMS, WCST, CPT, Trail Making, Stroop, BDI, and BRIEF (see [[concepts/neuropsychological-tests]]).

### 3. PHI De-identification

PHI de-identification in production clinical pipelines requires more than a single regex pass. The Neuropsychological Report RAG Pipeline describes a layered multi-stage approach implemented in `src/report_deidentify.py`:

1. **Strip page artifacts**: Remove CONFIDENTIAL headers, page numbers, and standalone name lines via `strip_page_artifacts()`.
2. **Name inference fallback chain**: When metadata extraction misses the patient name, `_infer_patient_name_from_text()` searches for "Name: First Last" patterns, CONFIDENTIAL footer formats, and standalone name lines. A further fallback (`_infer_patient_name_from_source_file()`) derives the name from the source filename.
3. **Build patient-specific replacement patterns**: `build_report_replacements()` generates name variants via `_name_forms()`, builds longest-first regex patterns to prevent partial matches, and adds date and case number patterns.
4. **Apply patient-specific replacements**: Each pattern replaces PHI with a **deterministic 6-character hash** via `_short_hash()` — ensuring the same patient maps to the same placeholder across multiple reports, supporting cross-report de-identified linking without re-identification risk.
5. **Apply general PHI safety nets**: `GENERAL_PHI_PATTERNS` (SSN, phone, email) and `REPORT_GENERAL_PHI_PATTERNS` (dates, MRN) catch anything missed by patient-specific patterns.

The deterministic hash approach is a key design choice: it enables consistent anonymization across a patient's multiple reports, while never exposing the original identifier. See [[concepts/phi-deidentification-pipeline]] for broader treatment.

The full pipeline entry points for de-identification in the RAG pipeline are:
- Patient-specific regex substitution: `src/report_deidentify.py:254`
- General PHI safety net patterns: `src/report_deidentify.py:258`

In assessment-heavy pipelines, de-identification also has a secondary methodological benefit: it supports cross-document synthesis without turning the retrieval layer into an uncontrolled repository of identifiable case material. That is especially important when pipelines incorporate recommendation retrieval, narrative drafting, or clinician-style matching.

### 4. Recommendation Section Parsing

Beyond general text chunking, the Neuropsychological Report RAG Pipeline implements a dedicated recommendation extraction layer in `src/report_parser.py`. This stage:

1. **Detects the RECOMMENDATIONS section boundary** — supporting variants like "Summary and Recommendations" — and finds the end boundary via signature blocks, appendices, or score tables.
2. **Filters non-actionable content** via `_filter_non_recommendation_content()`, removing clinical impressions that are not actionable recommendations.
3. **Detects subsection headers** using heuristics: PHASE X patterns, ALL CAPS patterns, and Title Case patterns.
4. **Builds `RecommendationChunk` objects** with text, sub-header, diagnoses list, age group, and clinical context metadata.

This metadata-rich chunking strategy enables post-retrieval filtering that plain text chunking cannot support — each chunk carries sufficient context to be filtered by diagnosis, age group, or clinical setting independently of the embedding similarity score.

The clinical-assessment lens strengthens the rationale for this design: recommendations are only meaningful when tied to population, setting, and problem context. For example, attention interventions may differ depending on whether difficulties are longstanding, context-specific, developmentally patterned, or plausibly acquired. Preserving those distinctions supports later reasoning related to [[concepts/premorbid-vs-acquired-attention-difficulties]], [[concepts/attention-intervention-strategies]], and [[concepts/developmental-vs-acquired-cognitive-symptoms]].

### 5. Chunk & Index
After extraction, raw text is chunked before embedding. The `chunk_text()` function applies a fixed-size windowing strategy:

```python
chunks = chunk_text(text, chunk_size=1200, overlap=150)
```

- **`chunk_size=1200` characters**: balances context preservation with embedding quality
- **`overlap=150` characters**: maintains continuity across chunk boundaries
- **Whitespace normalization**: collapses multiple spaces and newlines before chunking

The Autism RAG system uses a different parameterization — **800-word chunks with 120-word overlap** — reflecting the longer context windows suitable for research document Q&A as opposed to clinical report extraction. See [[concepts/text-chunking]] for broader discussion of chunking strategies and trade-offs.

The Neuropsychological Report RAG Pipeline uses **subsection-aware chunking** for clinical recommendations, implemented in `split_recommendations_by_subsection()`. Rather than fixed character windows, chunks are split at detected subsection headers using heuristics:
- ALL CAPS header patterns
- Title Case header patterns
- PHASE X patterns

Each `RecommendationChunk` carries structured metadata: diagnoses list, age group, clinical context, and sub-header. See [[concepts/rag-chunking]] for broader discussion.

Extracted structured data and raw text chunks are then stored for retrieval. The Luria Streamlit App uses a **dual-store** pattern:
- **SQLite** (`data/neuropsych.db`) for structured relational data: `Documents`, `ClinicalSummaries`, and `TestScores` tables (see [[concepts/sqlite-as-vector-store]])
- **LanceDB** (`data/vectors/`) for semantic vector search over narrative chunks (see [[concepts/lancedb-vector-store]])

This dual-storage pattern enables both precise SQL queries (filtering by `doc_id`, cognitive domain, test name) and fuzzy semantic retrieval — the foundation of [[concepts/retrieval-augmented-generation]] in clinical contexts. Optional local embeddings via oMLX replace cloud embedding providers entirely, reinforcing the local-first design (see [[concepts/local-llm-inference]]).

The Autism RAG and Neuropsychological Report RAG pipelines both use **FAISS IndexFlatIP** (see [[concepts/faiss-vector-index]]) with 384-dimensional vectors from the `all-MiniLM-L6-v2` SentenceTransformer model (see [[concepts/sentence-transformer-embeddings]]). The filtered search implementation (`search_filtered()` in `retrieval.py`) uses an **over-fetch strategy** (k × 3) to compensate for post-filter losses, applying list membership checks (e.g., for diagnoses) and equality checks (e.g., for age_group) as a post-retrieval pass. This is a practical pattern wherever metadata filtering must be layered on top of approximate nearest-neighbor search.

The full ingestion sub-pipeline for the SOUL style agent is:

```text
Report Files → extract_text() → chunk_text() → embed_with_fallback() → SQLite
```

### 6. Semantic Search with Metadata Filtering

The Neuropsychological Report RAG Pipeline introduces a dedicated search trace that goes beyond simple k-nearest-neighbor retrieval:

```text
User query → SentenceTransformer.encode() → VectorStore.search_filtered() → post-filter by diagnosis → ranked results
```

The `search_filtered()` method in `retrieval.py` applies a two-stage filter:
1. **Over-fetch**: Retrieve k × 3 candidates from FAISS using cosine similarity (inner product on normalized vectors).
2. **Metadata filter**: Check list membership (e.g., diagnoses field) and equality (e.g., age_group, context) on each candidate's metadata.

A further post-filter is applied in the UI layer (`app_recommendations.py`) for disorder selection — demonstrating that metadata filtering can be layered at both the retrieval layer and the application layer without duplicating index logic. The key entry points for semantic search in the RAG pipeline are:
- Query embedding: `app_recommendations.py:214` via `embeddings.py:19`
- FAISS similarity search: `src/retrieval.py:106`
- Post-filter by diagnosis list: `app_recommendations.py:230`

See [[concepts/vector-search]] for broader treatment of filtered vector retrieval.

Clinically, metadata filtering matters because recommendations, examples, and interpretations are not universally interchangeable. Pipelines should be able to retrieve material by diagnosis, age group, context, and assessment source, and in some cases by rater or instrument type. That is especially relevant for contexts involving [[concepts/substance-use-clinical-assessment]], [[concepts/trauma-informed-clinical-assessment]], or neurodevelopmental presentations with strong setting effects.

### 7. Generate
A language model synthesizes the indexed data into a new clinical artifact:
- Narrative summaries grouped by cognitive domain (see [[concepts/cognitive-domains]])
- Application of normative benchmarks and flagging of significant discrepancies
- Evidence-based recommendations
- Optional style matching from historical reports ("voice injection")

In the Luria Streamlit App, the report node generates a markdown narrative, with optional rendering via Typst CLI or Quarto for print-ready PDFs (see [[concepts/typst-typesetting]] and [[concepts/quarto]]). A further optional layer — **Luria Voice / SOUL** — injects clinician-specific style by retrieving exemplars from de-identified prior reports stored in a local SQLite index.

The Neuropsychological Report RAG Pipeline describes a multi-provider LLM generation layer in `src/llm.py`:

1. **Provider abstraction**: Providers (Anthropic, OpenAI, Ollama) are loaded dynamically via `importlib.import_module()` — avoiding unnecessary dependency imports when a provider is not selected. See [[concepts/llm-provider-abstraction]].
2. **Chunk formatting**: Retrieved `RecommendationChunk` objects are formatted with metadata (diagnoses, age, context, sub-header) as numbered examples: `"Example {i} (diagnoses...)"`. This grounds LLM output in real clinical practice patterns.
3. **Message construction**: A `SystemMessage` carries clinical guidelines and patient context; a `HumanMessage` carries the query, formatted examples, and generation instructions. This is a concrete instance of the system/human message split pattern in LangChain.
4. **LLM invocation**: `llm.invoke(messages)` returns `response.content` — the generated recommendation text.

The Autism RAG system expresses two distinct generation patterns:

1. **Research Q&A generation**: Uses **FLAN-T5** (`google/flan-t5-base`) for lightweight text-to-text generation over retrieved research document chunks.
2. **Clinical recommendation generation**: Uses the same [[concepts/llm-provider-abstraction]] layer supporting Anthropic Claude, OpenAI GPT, and Ollama.

This stage is handled by a separate narrative-writer agent — explicitly downstream of the data-extractor. The stage boundary is enforced by design: the data-extractor produces no interpretive language ("suggesting", "consistent with", "indicates"), leaving all clinical synthesis to the generate stage. See [[summaries/report-generation]] for more on report synthesis.

From the standpoint of [[summaries/clinical-assessment]], generation should also preserve uncertainty and context. Low cross-informant agreement, mixed symptom chronicity, or unclear onset should not be collapsed into overconfident prose. Strong systems preserve distinctions between converging evidence, diverging context-specific observations, and possible methodological artifacts.

#### Clipboard Output Patterns

When clinical NLP pipeline output is rendered in a web UI (e.g., Shiny for Python), delivering generated recommendations to end users often requires clipboard integration. Because clipboard access is a browser-only API (`navigator.clipboard`), it must be handled client-side via JavaScript injected through `ui.tags.script()`. Two copy formats serve distinct clinical workflows:

- **Copy Markdown** — pastes raw markdown text, ideal for markdown-aware editors (Notion, Obsidian, etc.) or text files
- **Copy HTML** — uses the `ClipboardItem` API with `text/html` MIME type, so pasting into Word, Google Docs, or email clients preserves all formatting (headings, numbered lists, bold text); a `text/plain` fallback ensures compatibility in plain-text contexts

Visual feedback is provided by briefly swapping Bootstrap classes (`btn-outline-secondary` → `btn-success`) and updating button text to "Copied!" for 2 seconds. See [[concepts/clipboard-api-patterns]] for implementation details.

## Pipeline Architecture

```text
pdf-parser → neuropsych-data-extractor → cingulate engine (DuckDB) → narrative-writer
```

The Luria Streamlit App expresses this as:

```text
parse_node → extract_node → index_node → report_node
```

The Autism RAG system uses two parallel pipelines sharing a common FAISS retrieval core:

```text
# Research Q&A
PDF/EPUB → chunk_text() → embeddings → FAISS → query → FLAN-T5 → answer + citations

# Clinical Recommendations
Reports → parse + de-identify → embeddings → FAISS → patient query → LLM → recommendations
```

The Neuropsychological Report RAG Pipeline similarly splits into ingestion and query paths:

```text
# Ingestion
PDF → fitz.open() → parse_report() → deidentify_recommendations() → SentenceTransformer → FAISS

# Query
User query → encode() → search_filtered() → post-filter → format_chunks() → LLM → recommendations
```

Each stage has a strictly scoped role:
- **pdf-parser / parse_node**: Ingest, redact PHI, normalize text — no interpretation
- **neuropsych-data-extractor / extract_node**: Convert clean text to structured data — no interpretation
- **index_node**: Store structured records in SQLite + LanceDB; chunk text for embedding
- **narrative-writer / report_node**: Synthesize clinical narrative — no raw data manipulation

A separate **RAG graph** (`build_rag_graph()`) provides a single-node retrieval path for the Ask tab, combining SQL filtering over `TestScores` with semantic search over narrative chunks. This modular split between the ingest pipeline and the retrieval pipeline is a key architectural decision in the Luria system.

The subagent system prompts for each node are sourced from `subagents/*/AGENTS.md` files — static prompt libraries that live in the repo and must be resolvable at pipeline startup (see [[concepts/subagent-architecture]]).

Architecturally, the most important clinical constraint is that representation choices upstream determine what kinds of assessment reasoning remain possible downstream. If the pipeline preserves domain, score type, rater, age group, diagnosis, and context, then retrieval and narrative generation can support clinically meaningful synthesis. If not, the system may still produce fluent text, but it will no longer support disciplined clinical assessment.

## Key Implementation Files

| File | Role |
|---|---|
| `src/ingest_recommendations.py` | Pipeline orchestrator (RAG pipeline) |
| `src/report_parser.py` | Diagnosis extraction and recommendation chunking |
| `src/report_deidentify.py` | PHI removal and de-identification |
| `src/retrieval.py` | FAISS vector store and filtered semantic search |
| `src/embeddings.py` | SentenceTransformer encoding |
| `src/llm.py` | LLM integration and prompt construction |
| `app_recommendations.py` | Shiny UI for search and generation |
| `soul/neuro_report_style_agent.py` | SOUL style agent with text extraction |

## User Interfaces for Clinical Pipelines

Clinical NLP pipelines commonly expose two interface types:

**Web UIs (Shiny for Python / Streamlit):** The Autism RAG system's Shiny app and the Neuropsychological Report RAG system's Streamlit app both provide reactive search interfaces where users submit queries and receive answer cards, citation cards with similarity scores, and Markdown export capability. The reactive `query_result.set(result)` pattern triggers UI re-renders without page reloads.

**REST APIs (FastAPI):** The Autism RAG system exposes `POST /query` for research Q&A with citations and `GET /ingest` to trigger document re-indexing. Pydantic models (`QueryRequest`, `QueryResponse`) provide schema validation at the API boundary.

Both interface types call the same underlying pipeline functions, enforcing the principle that pipeline logic is interface-agnostic.

For clinical users, interfaces should also make provenance visible: what was extracted, from which document, from which section, and when applicable from which informant or instrument. This is particularly valuable in assessment contexts where apparent contradictions are clinically meaningful rather than simple errors.

## Evaluation

Clinical NLP pipelines require systematic evaluation beyond ad hoc spot-checking. The Autism RAG system implements a YAML-based test suite:
- Each test case specifies a question and expected answer keywords
- `run_evaluation()` queries the full pipeline for each question
- `evaluate_answer()` applies keyword-matching heuristics and returns a score
- Results are averaged and written to JSON

The Neuropsychological Report RAG pipeline supports evaluation via `python eval/run_eval.py`. While keyword matching is a heuristic method, it provides a reproducible, automated baseline for detecting regressions. More sophisticated evaluation (semantic similarity, clinical expert review) can be layered on top. See [[concepts/yaml-configuration]] for the test suite format.

For assessment-oriented pipelines, evaluation should extend beyond answer fluency to include:
- preservation of score accuracy
- correct attribution of rater/source information
- correct handling of diagnosis normalization
- appropriate uncertainty when evidence conflicts
- resistance to criterion contamination or circular retrieval

These concerns reflect the realities described in [[summaries/clinical-assessment]], where discrepant reports and context-specific behavior are expected features of the data.

## Orchestration Patterns

Modern clinical NLP pipelines are typically orchestrated as **directed acyclic graphs (DAGs)** or **state machines**, not simple sequential scripts. This enables:
- Conditional branching (e.g., skip extraction if data already indexed)
- Parallel node execution
- State passing between nodes
- Retry and error handling per stage

The Luria system uses **LangGraph** — a StateGraph framework — to wire its pipeline nodes with explicit state transitions. The `PipelineState` TypedDict carries: accumulated messages, document identity (`pdf_path`, `doc_id`), stage outputs (`parsed`, `records`, `indexed`, `report`), RAG fields (`user_query`, `rag_answer`), and Luria Voice options (`voice_enabled`, `soul_db_path`, `quarto_format`). See [[concepts/langgraph-agent-workflows]] for broader coverage of this orchestration approach.

The parse worker's fail-loudly design principle — explicitly reporting inaccessible documents rather than producing empty or partial output — is essential here. Silent failures in early pipeline stages can propagate corrupted state through all downstream nodes, making explicit error surfacing a reliability requirement, not merely a convenience. The same principle applies at the code level: `extract_text()` raises explicit `ValueError` or `RuntimeError` exceptions rather than returning empty strings on failure.

### Path Resolution as a Reliability Concern

A subtle but critical operational issue for file-based pipelines (where node prompts are loaded from disk) is **static path resolution**. The Luria recovery (see [[summaries/RECOVERY_NOTES]]) illustrates this concretely: a workspace reorganization moved the Streamlit app into a subdirectory, but `nodes.py` still computed `REPO_ROOT` relative to its own file location — causing the extract node to crash with a `FileNotFoundError` before any document was processed.

Best practices to prevent this class of failure:
- Define distinct named constants for each root level (`APP_ROOT`, `WORKSPACE_ROOT`) rather than computing roots by traversing `__file__` parent chains.
- Add a **smoke test script** (like `scripts/smoke_test_paths.py` in the Luria repo) that resolves every static path and AST-parses every `.py` file without requiring heavy runtime dependencies. Run it before every demo or deployment.
- Verify paths in CI/CD so that workspace restructuring cannot silently break prompt loading.

This is also directly connected to [[concepts/monorepo-workspace-layout]]: multi-package workspaces (e.g., uv workspaces) introduce multiple notions of "root" that must be explicitly managed.

## Security & PHI in the Pipeline

The Luria Streamlit App's pipeline architecture makes a critical security guarantee: **PHI redaction is local-only**. The sequence is:
1. Docling parses the PDF locally.
2. PHI is redacted locally (replacing identifiers with tokens).
3. Only the redacted text is sent to Anthropic's Claude API for extraction.
4. All storage (SQLite, LanceDB) is local — no cloud vector store, no cloud database.
5. Audio transcription (MacWhisper) and summarization (oMLX) are also local.

The Autism RAG system applies the same principle at the ingestion layer via `deidentify_recommendations()`, which redacts PHI (names, dates, IDs) from clinical reports before embedding and storing them in FAISS. The Neuropsychological Report RAG Pipeline extends this with a **deterministic hash-based replacement** strategy: each patient's PHI is replaced with a consistent 6-character hash derived from their name, enabling cross-report de-identified linking while ensuring the vector index never contains identifiable patient data. The hash is generated by `_short_hash()` in `report_deidentify.py`, and is applied *before* embeddings are generated — so no PHI ever enters the vector index.

For the SOUL style agent, historical reports are similarly ingested locally via `extract_text()`, chunked, and embedded — with the resulting style index stored in SQLite on-device. This architecture means the pipeline can be used with real PHI while remaining HIPAA-conscious, provided the local environment is secured. See [[concepts/clinical-data-privacy]] and [[concepts/pii-redaction-pipelines]] for broader treatment of these concerns.

From a clinical-assessment standpoint, privacy is not only a compliance issue but a workflow enabler: it makes it possible to process rich, longitudinal, multi-source assessment material locally while retaining enough structure to support interpretation, report drafting, and recommendation retrieval.

## Related Documents
- [[summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_PIPELINE]]
- [[summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_CODEMAP]]
- [[summaries/neuropsych-data-extractor]]
- [[summaries/neuropsych-narrative-writer]]
- [[summaries/neuropsych-pdf-parser]]
- [[summaries/text-extraction]]
- [[summaries/README]]
- [[summaries/DEMO_GUIDE]]
- [[summaries/RECOVERY_NOTES]]
- [[summaries/AGENTS_luria]]
- [[summaries/AUTISM_RAG_SYSTEM_DOCUMENTATION]]
- [[summaries/DIAGNOSIS_FIX_SUMMARY]]
- [[summaries/DEPENDENCIES]]
- [[summaries/nse_narrative]]
- [[summaries/2026-02-11-this-session-is-being-continued-from-a-previous-co]]
- [[summaries/AGE_OVERRIDE_GUIDE]]
- [[summaries/DIAGNOSIS_PARSER_IMPROVEMENTS]]
- [[summaries/report-generation]]
- [[summaries/clinical-assessment]]

See also: [[summaries/RECOMMENDATION_FILTERING_IMPROVEMENTS]]