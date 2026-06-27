---
sources: [summaries/PERMANENT_SOLUTION_SUMMARY.md, summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_PIPELINE.md, summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_CODEMAP.md, summaries/DIAGNOSIS_PARSER_IMPROVEMENTS.md, summaries/DIAGNOSIS_FIX_SUMMARY.md, summaries/AGE_OVERRIDE_GUIDE.md, summaries/extract_pdf.md, summaries/CLAUDE.md, summaries/2026-04-26-cingulate-agent-team-design.md, summaries/agent-team.md, summaries/full-pipeline.md, summaries/report-rendering-pipeline.md, summaries/text-extraction.md, summaries/report-generator.md, summaries/0010-voice-quarto-typst-reporting.md, summaries/multi_patient_transcript.md, summaries/report_body.md, summaries/NP-20240415-001_report.md, summaries/SESSION_SUMMARY_2025-04-28.md, summaries/RECOVERY_NOTES.md, summaries/PROJECT_SETUP_COMPLETE.md, summaries/neuropsych-pdf-parser.md, summaries/neuropsych-narrative-writer.md, summaries/neuropsych-data-extractor.md, summaries/clinical-validity-reviewer.md, summaries/responses_to_claude.md, summaries/processed_files.md, summaries/conversation-export.md, summaries/WORKFLOW_INSTRUCTIONS.md, summaries/TECHNICAL_DOCS.md, summaries/SHINY_APP_FIXED.md, summaries/REBUILD_FINAL_STATUS.md, summaries/REBUILD_COMPLETE.md, summaries/README_WORKFLOW.md, summaries/README_PIPELINE.md, summaries/README_AS_PROCESSING.md, summaries/QUICK_REFERENCE.md, summaries/OCR_PDF_GUIDE.md, summaries/KNOWLEDGE_BASE_EXPLAINED.md, summaries/FIX_EXPLANATION.md, summaries/EMBEDDINGS_COMPLETE.md, summaries/AS_PROCESSING_COMPLETE.md, summaries/brainworkup-brand-voice-guide.md, summaries/report-generation.md, summaries/mcp-integration.md, summaries/template-system.md, summaries/quarto-extensions.md, summaries/overview.md, summaries/003-modular-template-structure.md, summaries/002-mcp-llm-integration.md, summaries/SKILL.md, summaries/README.md, summaries/project-setup-progress.md, summaries/AGENTS_luria.md]
brief: Multi-stage workflow converting raw neuropsychological PDFs into structured scores, narratives, and formatted clinical reports.
---

# Neuropsychological Assessment Pipeline

A **neuropsychological assessment pipeline** is a structured, multi-stage workflow in which specialized agents or processing components handle distinct phases of neuropsychological document analysis — from raw PDF ingestion through scoring, interpretation, narrative generation, and final report rendering. This architecture separates concerns cleanly, ensuring that each stage receives well-formed input and produces reliable output for the next.

## Why a Pipeline Architecture?

Neuropsychological documents are complex: they combine structured numerical data (test scores, percentiles, T-scores), narrative clinical text, patient metadata, and domain-specific terminology. A single monolithic processor struggles with this heterogeneity. A pipeline approach assigns each concern to a dedicated stage, improving reliability, maintainability, and auditability.

This pattern is closely related to [[concepts/multi-agent-orchestration]] and [[concepts/langgraph-agent-workflows]], where agents are chained or orchestrated to handle sub-tasks in sequence or in parallel.

## Implementations

The pipeline concept is realized in two related systems:

### Luria (Primary System)
All pipeline processing is routed through the **`engine/cingulate`** engine (located at `~/luria/engine/cingulate`), which replaced an earlier Google Sheets integration. Key design decisions:

- **Data storage formats:** CSV, Arrow, and Parquet — not spreadsheets or XLSX files.
- **Report generation:** [[concepts/quarto]] and [[concepts/typst-typesetting]] pipeline.
- **No Google Sheets dependency:** Removed approximately one month prior to the current version; the `Sheets_Data_Indexer` subagent spec is now effectively legacy documentation.

The folder was renamed from `R/cingulate` to `engine/cingulate` under `~/luria`. See [[concepts/cingulate-engine]] for the engine's design rationale.

### Luria Streamlit App (Local-First Desktop Implementation)
A parallel implementation wraps the same conceptual pipeline stages in a Streamlit desktop UI with a LangGraph orchestrator. This implementation is explicitly designed for HIPAA-conscious solo clinicians who need real PHI to stay on their own machine. Key characteristics:

- **UI**: Streamlit at `http://127.0.0.1:8501`, exposing four tabs: Ingest, Ask, Knowledge Base, and Audio.
- **Orchestration**: LangGraph `StateGraph` with nodes for parse, extract, index, report, and RAG retrieval.
- **Local-first**: PDF parsing, embeddings, vector search, and structured storage all run on-device. Only the extraction step calls Anthropic's API (Claude Sonnet), and only after local PHI redaction via Docling.
- **Dual stores**: SQLite (`data/neuropsych.db`) for relational data; LanceDB (`data/vectors/`) for semantic retrieval of narrative chunks.
- **Optional local LLM**: oMLX (OpenAI-compatible local server) for chat, summarization, and embeddings.
- **Audio**: MacWhisper CLI for local transcription; oMLX for local summarization.
- **Report rendering**: Optional Typst or Quarto rendering for print-ready PDFs.

The Streamlit app's `PipelineState` TypedDict carries: `messages`, `pdf_path`, `doc_id`, `parsed`, `records`, `indexed`, `report`, `user_query`, `rag_answer`, and Luria Voice options (`voice_enabled`, `soul_db_path`, `soul_profile_path`, `quarto_format`).

See [[summaries/README]] for the full Streamlit app specification.

## Subagent Specifications

The pipeline defines four subagent specs under `subagents/`:
- `subagents/Neuropsych_Data_Extractor/`
- `subagents/PDF_Ingestion_Parser/`
- `subagents/Narrative_Report_Generator/`
- `subagents/Sheets_Data_Indexer/` *(legacy — Google Sheets integration removed)*

Each spec consists of an `AGENTS.md` + `tools.json` pair. These are currently **documentation**, not dispatchable subagents. Promoting them to `.claude/agents/<name>.md` with appropriate frontmatter would allow the Task tool to invoke them in parallel for a full intake → extraction → narrative pass. In the Streamlit app, these same `AGENTS.md` files serve directly as LangGraph system prompts without modification. See [[concepts/subagent-architecture]] for the broader pattern.

## Stages of the Pipeline

### Stage 0 — Patient Metadata & Age Classification

Before document ingestion begins, the pipeline must establish patient metadata — most critically, `age_group`, which governs which norms, narrative register, and report template are applied downstream. The [[concepts/age-group-classification]] system maps numeric ages to four groups:

| Age Range | Group |
|---|---|
| < 13 | `pediatric` |
| 13–17 | `adolescent` |
| 18–64 | `adult` |
| 65+ | `geriatric` |

Age is extracted automatically via `src/report_parser.py` using a fallback chain:
1. **Header pattern** — e.g., `PATIENT NAME: John Doe, Age 25`
2. **Body text pattern** — e.g., "is a 25-year-old" or "25 year old patient"
3. **Age Override File** — `data/age_overrides.json`, consulted when automatic extraction fails

This override mechanism is a permanent solution for PDFs that lack extractable age information. The override file stores filename-to-age mappings:

```json
{
  "overrides": {
    "report_filename.pdf": 42,
    "another_report.pdf": 15
  }
}
```

Overrides persist across re-ingestions and are transparent — all corrections are visible in a single file. Key operational notes:
- Filenames are **case-sensitive**
- The file must reside at `data/age_overrides.json` relative to project root
- Re-running `python -m src.ingest_recommendations` is required for changes to take effect
- If the PDF itself contains an age, that takes precedence over any override entry
- Two reports remain `age_group: unknown` by design (duplicates/templates without patient info): `autumn_w_summer_data.pdf` and `donders__neuropsychological_report_writing_examplereport_2.pdf`

See [[summaries/AGE_OVERRIDE_GUIDE]] for the full override system specification.

After ingestion, unknown age groups can be audited:

```python
import json
with open("data/recommendations_kb.json") as f:
    data = json.load(f)
for report in data["metadata"]["reports"]:
    if report["age_group"] == "unknown":
        print(f"Unknown age: {report['source_file']}")
```

### Stage 1 — Ingestion & Parsing (neuropsych-pdf-parser)

The entry point of the pipeline is the **neuropsych-pdf-parser**, a dedicated Stage 1 agent whose sole responsibility is turning a raw PDF into clean, structured, PHI-free text. It explicitly does **not** interpret, score, or summarize — all interpretation is delegated to later pipeline stages.

In the Streamlit implementation, this stage is handled by the **Docling** PDF parser (running entirely locally), which extracts text and layout before any data is sent anywhere. See [[concepts/docling-pdf-parsing]] for details on this local extraction approach.

#### Text Extraction Module

Before any document classification or PHI scrubbing, raw file content must be read into memory. This responsibility belongs to the `extract_text(path: Path) -> str` function, implemented in `soul/neuro_report_style_agent.py` (lines 54–67). It supports two file formats:

| Format | Extension | Implementation |
|--------|-----------|----------------|
| Plain Text | `.txt`, `.md` | Native Python `path.read_text()` |
| PDF | `.pdf` | PyPDF2 `PdfReader` |

Error handling:
- **Unsupported file type**: Raises `ValueError` — `f"Unsupported file type: {path}"`
- **Missing PyPDF2**: Raises `RuntimeError` with installation instructions
- **Encoding issues**: Falls back to UTF-8 with `errors="ignore"`

File discovery is handled by `iter_report_files()`, a generator that recursively finds all matching files using `rglob` with patterns `*.pdf`, `*.txt`, and `*.md`.

See [[summaries/text-extraction]] for the full module specification.

#### Text Chunking

After raw text extraction, content is passed through `chunk_text()` before embedding. This is an early instance of [[concepts/text-chunking]] in the pipeline:

```python
chunks = chunk_text(text, chunk_size=1200, overlap=150)
```

- **`chunk_size=1200`**: Balances context preservation with embedding quality
- **`overlap=150`**: Maintains continuity across chunk boundaries
- **Whitespace normalization**: Collapses multiple spaces/newlines before chunking

The chunked output feeds directly into the embedding step (`embed_with_fallback()`) and then into SQLite, completing the local ingestion sub-pipeline:

```
Report Files → extract_text() → chunk_text() → embed_with_fallback() → SQLite
```

#### Input

The parser accepts:
- An **absolute local PDF path** (preferred) — uses `Read` for short documents, `PageIndex` tools (`process_document`, `get_document_structure`, `get_page_content`) for PDFs longer than 20 pages.
- A **URL** — uses `WebFetch` for HTML, or downloads before processing.

#### Workflow

1. **Source identification** — determines local vs. remote and selects the appropriate tool.
2. **Long-document handling** — for PDFs >20 pages, calls `process_document` → `get_document_structure` → `get_page_content` on targeted page ranges. Avoids blind full-document reads.
3. **Document classification** — assigns exactly one of five types:
   - `neuropsych_assessment_report` (WAIS, WIAT, WMS, RBANS, MoCA, MMSE, CPT, BRIEF, etc.)
   - `psychometric_score_sheet` (raw data, normative tables)
   - `clinical_notes` (progress notes, intake, history)
   - `research_paper` (peer-reviewed or preprint)
   - `mixed_other` (with specified subtype)
4. **Text extraction** — preserves section headings, tables, lists, and all numerical values character-for-character (raw scores, scaled scores, T-scores, standard scores, percentiles, SDs).
5. **PHI scrubbing** — aggressive replacement of all identifying information.

On extraction failure, the parser returns `DOCUMENT_TYPE: extraction_failed` with a one-line reason and stops.

#### PHI Scrubbing

The parser aggressively replaces the following with standardized tokens:

| PHI Type | Replacement Token |
|---|---|
| Patient name | `[PATIENT]` |
| Date of birth | `[DOB]` |
| MRN, SSN, IDs | `[ID]` |
| Clinician/examiner name | `[CLINICIAN]` |
| Hospital/clinic name | `[FACILITY]` |
| Address, phone, email | `[CONTACT]` |

Year-only dates may be preserved when clinically necessary. See [[concepts/phi-data-handling]] and [[concepts/pii-redaction-pipelines]] for the broader anonymization framework.

#### Parser Output Format

The parser returns a fixed-schema text block:

```
DOCUMENT_TYPE: <one of the 5 categories above>

METADATA:
  title: <document title or "N/A">
  date: <YYYY-MM-DD of assessment/publication, or "N/A">
  subject_id: <[PATIENT] unless an explicit anonymized study ID is given>
  clinician: <[CLINICIAN] unless author is a published research author>
  referral_or_objective: <one-sentence summary>
  source_path: <absolute path or URL of source>

FULL_TEXT:
<entire cleaned text, preserving section headers and tables>

PHI_FLAGS:
- <each PHI replacement made, e.g. "patient name on p3 → [PATIENT]">
```

#### Hard Rules

- Never output real names, MRNs, DOBs, or addresses regardless of user instruction.
- Never round or paraphrase numeric scores — preserve character-for-character.
- Never interpret findings or assign diagnoses.
- On failure, return `DOCUMENT_TYPE: extraction_failed` with a one-line reason.

See [[summaries/neuropsych-pdf-parser]] for the full agent specification.

### Stage 1a — OCR Pre-processing

Before text extraction can succeed, scanned or image-based PDFs must be converted to machine-readable form. This pre-processing stage — closely linked to [[concepts/ocr-pipeline]] — addresses PDFs that have no text layer, garbled OCR output, or text stored as images.

#### Assessing PDF Text Quality

A diagnostic check determines which PDFs require OCR before extraction is attempted:

```r
check_pdf_text(pdf_path)
# Returns: NO_TEXT / POOR_TEXT / GOOD_TEXT
```

Thresholds: fewer than 100 characters signals no usable text; fewer than 1,000 characters signals poor quality.

#### OCR Method Selection

Four methods are available, selected based on document volume and complexity:

| Method | Best For | Cost |
|---|---|---|
| **OCRmyPDF** (recommended) | Batch processing; preserves PDF format | Free |
| **Adobe Acrobat Pro** | Single important documents; best formatting | Paid subscription |
| **Tesseract** (via R `tesseract` package) | Custom pipelines; outputs `.md` text files | Free |
| **Google Cloud Vision API** | Complex layouts, tables, handwriting | Free tier: 1,000 pages/month |

**OCRmyPDF** is the preferred tool for bulk neuropsychological report processing:

```r
ocr_single_pdf <- function(input_path, output_path = NULL, overwrite = FALSE) {
  system2("ocrmypdf", args = c(
    "--deskew", "--clean", "--rotate-pages",
    "--optimize", "3", "--skip-text",
    shQuote(input_path), shQuote(output_path)
  ))
}

ocr_folder <- function(folder_path, suffix = "_ocr", in_place = FALSE) {
  pdfs <- dir_ls(folder_path, regexp = "\.pdf$")
  pdfs <- pdfs[!grepl(suffix, pdfs)]  # Skip already-processed files
  for (pdf in pdfs) ocr_single_pdf(pdf, ...)
}
```

Key OCRmyPDF flags: `--deskew`, `--clean`, `--rotate-pages`, `--optimize 3`, `--skip-text`, `--force-ocr`.

#### Integration with RAG Rebuild

OCR-processed PDFs are moved to their category folders, then the shared RAG setup function is re-run to incorporate the new content into the [[concepts/retrieval-augmented-generation]] index.

See [[summaries/OCR_PDF_GUIDE]] for the complete OCR utilities reference.

### Stage 2 — Score Extraction (neuropsych-data-extractor)

Stage 2 is implemented by the **neuropsych-data-extractor**, which operates on the clean, parsed, PHI-scrubbed text produced by Stage 1. Its sole responsibility is converting that text into **long-format CSV rows** consumable by the cingulate engine via DuckDB's `read_csv_auto` loader.

In the Streamlit app, this stage is handled by a LangGraph node that invokes Claude Sonnet (Anthropic cloud) to structure the parsed narrative into JSON containing test scores and clinical summaries. This is the only stage that makes a network call — and only after PHI has been redacted locally.

#### A Concrete Example: NP-20240415-001

The report generated for case **NP-20240415-001** (see [[summaries/NP-20240415-001_report]] and [[summaries/report_body]]) illustrates both the pipeline's capabilities and its limitations when only a single subtest is administered. The entire evaluation for that case rested on the **WAIS-IV Digit Span** subtest:

| Metric | Value |
|---|---|
| Raw Score | 16 |
| Scaled Score | 9 |
| Percentile Rank | 37th |
| Classification | Average |
| Composite Index | Working Memory Index |

The extractor mapped this single measurement across six cognitive domains (Intelligence, Memory & Learning, Attention & Processing Speed, Executive Function, Language, Visuospatial Ability) — a structurally valid but clinically limited approach, since all domain interpretations derived from a single working memory measure. This case illustrates why comprehensive assessment batteries are essential: a pipeline built for breadth cannot compensate for a narrow test administration.

The report also reveals a **classification inconsistency** worth noting as a QA consideration: the `range` column recorded "Average" while the clinical significance narrative stated "below average (≤16th percentile)". This discrepancy between a classification label and percentile-derived clinical significance is a known artifact when scaled score 9 (37th percentile) is labeled "Average" by normative tables yet clinicians apply a stricter clinical threshold. Downstream narrative-writer prompts should be designed to resolve such contradictions explicitly rather than propagating them into the final report.

#### Critical Format Rule: Long Format Only

The cingulate engine uses **long format**, not wide format. Every test/subtest measurement is exactly **one row**. Score-type variants (scaled, T, standard, z, base-rate) live in a `score_type` column — they are never spread across separate columns. See [[concepts/long-format-clinical-data]] for the general pattern.

#### Canonical Schema

The extractor writes the following columns:

| Column | Description |
|---|---|
| `test` | Full test/subtest identifier, e.g. "WAIS-IV Digit Span" |
| `test_name` | Short label used in tables |
| `scale` | Composite or index name, e.g. "Working Memory Index" |
| `raw_score` | Numeric raw score (NA if not reported) |
| `score` | Numeric standardized score value |
| `score_type` | One of: `scaled_score`, `t_score`, `standard_score`, `z_score`, `base_rate` |
| `percentile` | Numeric percentile rank 0–99 (NA if not reported) |
| `range` | Qualitative classification, e.g. "Average", "Impaired" |
| `domain` | `Neuropsychological Test Score`, `Behavioral/Emotional/Social`, or `Effort/Validity Test` |
| `subdomain` | Cognitive subdomain, e.g. "Working Memory", "Processing Speed" |
| `narrow` | Narrow construct under subdomain |
| `pass` | PASS theory tag or NA |
| `verbal` | `Verbal` or `Nonverbal` |
| `timed` | `Timed` or `Untimed` |
| `description` | One-sentence test description |
| `rater` | `self`, `parent`, `teacher`, or `examiner` |
| `age_group` | `child`, `adolescent`, or `adult` — populated from Stage 0 classification |
| `doc_id` | Source document ID from the parser |
| `date` | Assessment date YYYY-MM-DD |

Required fields (enforced by `domain_processing_utils.R:991`): `domain`, `rater`, `age_group`, `test`.

The **Streamlit app's SQLite schema** reflects these same concepts with a `TestScores` table containing: `test_name`, `subtest_name`, `scaled_score`, `standard_score`, `t_score`, `percentile_rank`, `classification`, `cognitive_domains_affected`.

See [[summaries/neuropsych-data-extractor]] for the full agent specification.

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

#### Row Decomposition Rules

- WAIS-IV typically yields ~20 rows (10 subtests × scaled score + composites).
- Multi-rater behavioral rating scales (BRIEF, CBCL, BASC, BDEFS) yield one row **per rater per scale**.
- Values are preserved exactly — no rounding, inference, or type conversion.
- Any value that cannot be determined is recorded as NA; no values are invented.
- PHI that slipped through Stage 1 scrubbing is replaced with `[PATIENT]`/`[CLINICIAN]` and flagged as a WARNING.

#### PDF Assessment & Instrument-Specific Logic

A critical lesson: **assessing whether a PDF contains text is not the same as assessing whether it contains extractable scores.** This distinction drives instrument-specific assessment before extraction is attempted.

For instruments like the PAI, T-scores may be embedded exclusively in graphical bar charts. A PAI-specific assessment function (`assess_pai_pdf()`) answers four questions:
1. Does the PDF have a text layer?
2. Is it a PAI report?
3. Are PAI T-scores present in the text layer (vs. only in graphics)?
4. What extraction method is appropriate, and is manual entry required?

This design is captured in [[concepts/pdf-score-extraction]] and [[concepts/clinical-pdf-assessment]]. Three outcome categories are recognized:
- **`TABLE_EXTRACTION`** — T-scores are in text/table form; automatic extraction is possible.
- **`GRAPHICAL_ONLY`** — T-scores are embedded in bar charts; manual entry is required.
- **`UNKNOWN`** — Format cannot be determined; manual review needed.

#### LLM-Assisted Extraction
For PDFs where text extraction is feasible, raw PDFs placed in `data/raw/pdf/` are processed by a Python script that invokes a [[concepts/model-context-protocol]] (MCP) server. The MCP server delegates to a local LLM backend — typically [[concepts/local-llm-inference]] via Ollama running a model such as `llama3.1` — to parse the PDF, extract structured test scores and metadata, and apply clinical terminology via a lookup table. Results are saved as structured JSON.

This stage is configured via [[concepts/yaml-configuration]] entries in `config.yml`:
```yaml
mcp:
  llm_base_url: "http://localhost:11434/v1"
  llm_model: "ollama/llama3.1"
  pdf_path: "data/raw/pdf/wisc5.pdf"
  lookup_table: "~/Dropbox/neuropsych_lookup_table.csv"
```

See [[summaries/002-mcp-llm-integration]] and [[summaries/mcp-integration]] for further detail on the MCP integration.

#### Extractor Output

The extractor writes a UTF-8 comma-separated CSV to `data-raw/csv/<doc_id>_neuropsych.csv` and verifies it with shell commands (`wc -l`, `head -3`). Its final message reports:
- `CSV_PATH`, `ROW_COUNT`, `DOMAINS_DETECTED`, `RATERS`, `SCORE_TYPES`, and `WARNINGS`.

### Stage 3 — Narrative Writing (neuropsych-narrative-writer)

Stage 3 is implemented by the **neuropsych-narrative-writer**, which operates on the long-format CSV produced by the extractor. Its sole responsibility is generating **per-domain prose narratives** as Quarto include files (`.qmd`) that drop into cingulate's existing template structure.

In the Streamlit app, this corresponds to the **report node** which generates a markdown narrative report, with optional Typst or Quarto rendering for print-ready PDFs. The NP-20240415-001 case (see [[summaries/report_body]]) demonstrates the narrative output structure: the writer produced six domain sections all drawing from the same Digit Span result, with a cognitive profile summary, clinical impressions, recommendations (neuroimaging, reassessment, cognitive rehabilitation referral), and a limitations section disclosing the single-test scope.

#### Output Files

For each cognitive domain present in the CSV, the agent writes a file named `_NN-XX_<domain>_text.qmd`. Prefixes must **never be renumbered** — `_quarto.yml` and `template.qmd` depend on the stable ordering.

#### Narrative Workflow

For each subdomain, the agent drafts **2–4 short paragraphs** covering:
- **Performance summary** — what was tested; qualitative range drawn verbatim from the `range` column.
- **Pattern interpretation** — relative strengths/weaknesses, intra-test scatter, score-type discrepancies.
- **Functional implication** — one hedged sentence linking to everyday/academic/occupational impact.

#### Edit-Protection

Before overwriting any existing `_text.qmd`, the agent reads the file first. If clinician hand-edits are detected, the agent appends its new draft as a `<!-- DRAFT: ... -->` comment block rather than overwriting. This implements the [[concepts/edit-protection-pattern]] designed to protect manual clinical edits.

#### Voice and Style

- Professional, APA-style neuropsychological report register.
- Hedged language: *"performance is consistent with…"*, *"results suggest…"*, *"indicates relative weakness in…"*.
- Adapts register to the `age_group` column: pediatric language for child reports, adult language for adult reports, forensic register for forensic evaluations.
- Markdown only — no raw HTML; Quarto-compatible.

See [[summaries/neuropsych-narrative-writer]] for the full agent specification.

### Stage 4 — Score Normalization
Downstream workers parse the structured CSV/JSON to identify and normalize test scores:
- Raw scores, scaled scores, T-scores, percentile ranks, standard deviations.
- Subtest-level and composite-level data.
- Instrument identification (WAIS-IV, MoCA, RBANS, Trail Making, Stroop, WCST, BRIEF, PAI, etc.).

For instruments like the PAI where scores may be graphical, the pipeline routes to a manual entry pathway rather than attempting automated text parsing.

See [[concepts/neuropsychological-test-scores]] and [[concepts/neuropsychological-tests]] for the domain vocabulary involved.

### Stage 5 — Data Processing (engine/cingulate)
Structured CSV/JSON is loaded into R, where the cingulate engine applies age-appropriate norms, calculates standard scores and percentiles, generates visualizations via ggplot2, and creates summary tables. Data is persisted in **CSV, Arrow, and Parquet** formats (not spreadsheets). This stage supports parallel processing and DuckDB integration for performance. Configuration is managed through `config.yml`:
```yaml
processing:
  use_duckdb: yes
  parallel: yes
  age_group: ""
```

See [[concepts/r-python-integration]] and [[concepts/r-visualization-theming]] for the R tooling involved.

### Stage 6 — Data Persistence & Indexing
Once structured records are produced, the pipeline persists all data into local storage. This stage serves two complementary functions:

**Relational storage (SQLite):**
- A `TestScores` table stores granular psychometric data — one row per test/subtest — including raw scores, scaled scores (1–19), standard scores (100 ± 15), T-scores (50 ± 10), percentile ranks, classifications, composite index labels, normative comparison notes, diagnoses, clinical impressions, and recommendations.
- A `ClinicalSummaries` table maintains one aggregate row per document, capturing high-level clinical context: diagnosis, affected cognitive domains, notable findings, and recommendations.
- A `Documents` table (added in the Streamlit app implementation) registers one row per ingested PDF with `doc_id`, `filename`, and `ingested_at`.

**Vector store:**
- Narrative text is chunked by section headers and embedded using a sentence-transformer model.
- In the Streamlit app, chunks are stored in **LanceDB** (`data/vectors/`) with `doc_id` and `chunk_id` metadata, enabling semantic search over clinical narratives. See [[concepts/lancedb-vector-store]].
- In the PAI sub-pipeline, **DuckDB** serves as the vector store backend. See [[concepts/duckdb-as-vector-store]].
- Persistence rules: idempotent upserts; append-only for test-score rows; exact data fidelity.

See [[summaries/AGENTS_luria]] for the full specification of the pipeline worker agents.

### Stage 7 — Domain Mapping & Analysis
Scores are mapped onto [[concepts/cognitive-domains]] (e.g., memory, attention, executive function, language, visuospatial ability). This stage may involve:
- Comparison to normative databases.
- Pattern recognition across subtests.
- Flagging of significant discrepancies or impairments.

This stage is where [[concepts/clinical-nlp-pipelines]] and [[concepts/retrieval-augmented-generation]] techniques may be applied to support interpretation with reference literature, drawing on the indexed vector store built in Stage 6.

### Stage 8 — Template Rendering & Report Generation
Findings are synthesized into a professionally formatted clinical report through two sub-stages:

**Quarto template rendering** ([[concepts/quarto]], [[concepts/quarto-extensions]]):
- `template.qmd` loads processed data, substitutes variables from `_variables.yml`, includes ordered section files, executes R code chunks, and produces an intermediate Typst markup file.
- Three report formats are supported: pediatric, adult, and forensic, selected via Quarto extension.
- Patient variables (name, DOB, age, sex, diagnoses, pronouns) are defined in `_variables.yml` and injected via `{{< var key >}}` shortcodes into QMD narrative content, R chunk contexts, and raw Typst blocks (`` ```{=typst} `` chunks), making the same template reusable across patients by editing a single YAML file.
- The section include chain is: tests administered → neurological status exam and behavioral observations → neurocognitive findings (dispatched dynamically) → summary/impressions, recommendations, signature, appendix. Include file prefixes are stable and must never be renumbered.
- The R setup chunk calls `pick_font()` to detect available system fonts and configures `svglite` + `ggplot2` so SVG figures use the same typeface as the Typst document body, ensuring visual consistency.

**Typst compilation** ([[concepts/typst-typesetting]]):
- The Typst compiler applies the selected extension template and styling rules to produce the final PDF.
- In the Streamlit app, the Typst CLI wrapper is at `neuropsych_agent/tools/typst_compiler.py`.
- Three format profiles are available: `neurotyp-pediatric-typst` (Equity B font), `neurotyp-adult-typst` (Libertinus Serif/Sans), and `neurotyp-forensic-typst` (Libertinus Serif/Sans). Each maps to a corresponding extension under `style/_extensions/brainworkup/`.
- Intermediate Typst source is retained when `keep-typ: true`; intermediate Markdown when `keep-md: true`.

**Prerequisites for rendering:**
- **Quarto** ≥1.4.0 (for Typst format support)
- **Typst** (bundled with Quarto or standalone)
- **R** with packages: `neuro2`, `dplyr`, `readr`, `here`, `yaml`, `ggplot2`, `svglite`
- **System fonts**: Equity B (pediatric), Libertinus Serif/Sans (adult/forensic)

**Luria Voice Integration** (optional, Streamlit app):
- **BRAND**: `_brand.yml` for logos, colors, typography.
- **SOUL**: Style profile + exemplar RAG from de-identified prior reports.
- **STYLE**: Quarto `neurotyp-*-typst` formats (adult, pediatric, forensic).
- Activated via `VOICE_ROOT`, `NEUROPSYCH_SOUL_DB`, and `NEUROPSYCH_SOUL_PROFILE` environment variables.

See [[concepts/narrative-report-generation]], [[concepts/neuropsychological-reporting]], [[summaries/report-generation]], [[summaries/003-modular-template-structure]], and [[summaries/template-system]] for output conventions and template architecture. See [[summaries/report-rendering-pipeline]] for the complete step-by-step rendering workflow reference.

---

## Pipeline Stage Summary

```
age classification / override lookup (Stage 0)
  → neuropsych-pdf-parser (Stage 1)
  → extract_text() + chunk_text() (Stage 1, text extraction sub-pipeline)
  → OCR pre-processing if needed (Stage 1a)
  → neuropsych-data-extractor → long-format CSV (Stage 2)
  → neuropsych-narrative-writer → per-domain _text.qmd files (Stage 3)
  → score normalization (Stage 4)
  → cingulate engine / DuckDB (Stage 5)
  → data persistence / vector store (Stage 6)
  → domain mapping & analysis (Stage 7)
  → Quarto + Typst report rendering (Stage 8)
```

The neuropsych-pdf-parser enforces a strict separation at Stage 1: it is only an extraction and scrubbing agent — not an interpreter. The neuropsych-data-extractor operates strictly between the parser and the cingulate engine, ensuring score data arrives in the exact schema the engine expects. The narrative-writer, in turn, produces only prose — tables, plots, and final assembly belong to the R6/cingulate layer.

---

## RAG Q&A Layer

Both implementations provide a retrieval-augmented generation layer for querying the indexed knowledge base:

- **Streamlit app Ask tab**: Chat interface querying only ingested documents. Uses SQL filtering over `TestScores` and semantic search over narrative chunks in LanceDB. Implemented as a single-node `build_rag_graph()` in LangGraph.
- **Luria PAI sub-pipeline**: `interpret_pai_from_manual_scores()` queries the PAI knowledge base and passes retrieved context to an LLM for narrative interpretation.
- **Honcho AI integration** (optional, Streamlit app): Peer observation via the Honcho SDK for session-based user modeling.

---

## PAI-Specific Pipeline Extension

The [[concepts/pai-assessment]] (Personality Assessment Inventory) represents a specialized application of the pipeline, extending it with a dedicated semantic retrieval layer, a careful PDF assessment step, and an AI-powered interpretation workflow driven by R scripts.

### Workspace and File Organization

The PAI sub-pipeline operates against a structured workspace:
- **`reports/` folder:** 79 PAI PDF files (`pai_00.pdf` through `pai_318.pdf`)
- **`source/` folder:** 19 research papers and PAI documentation, prefixed `pai_source_`
- **`input/` folder:** New patient files (e.g., `AS_PAI_Report.pdf`, `AS_scores_template.json`)
- **Total knowledge base:** 98 PDFs

### Three-Step Operator Workflow

**Step 1 — Rebuild Knowledge Base**
```r
source("/Users/joey/rag/pai/rebuild_pai_ragnar.R")
```

**Step 2 — Extract PAI T-Scores**
Two pathways exist:
- **Option A:** If a Summary Table PDF is present, place it in `input/` and extract tabular T-scores directly.
- **Option B (most common):** Open the PAI Report PDF, go to pages 3–4 (PAI Full Scale Profile), read T-scores from bar graphs, and fill in `input/AS_scores_template.json`.

**Step 3 — Generate Interpretation Report**
```r
source("/Users/joey/rag/pai/generate_as_interpretation.R")
```

### PDF Format Challenge
PAI T-scores are frequently **not present in the text layer** of PAI report PDFs. They are embedded as graphical bar charts (typically on pages 3–4 of a Score Report). When graphical-only format is detected, OCR pre-processing (Stage 1a) cannot rescue the situation — graphical bar charts encode score values visually, not as text. Manual entry remains the only reliable pathway.

See [[summaries/FIX_EXPLANATION]] for the full diagnosis and fix.

### PAI Scales Covered

| Domain | Scales |
|---|---|
| Validity | ICN, INF, NIM, PIM |
| Clinical | SOM, ANX, ARD, DEP, MAN, PAR, SCZ, BOR, ANT, ALC, DRG |
| Treatment | AGG, SUI, STR, NON, RXR |
| Interpersonal | DOM, WRM |

### AI-Powered Interpretation via RAG
Once T-scores are entered, the pipeline calls `interpret_pai_from_manual_scores()`, which queries the [[concepts/pai-knowledge-base]] and passes retrieved context to an LLM:

```r
result <- interpret_pai_from_manual_scores(
  scores      = patient_scores,
  con         = con,
  provider    = "ollama",
  model       = "llama3.2",
  output_file = "output/PAI_Interpretation.txt",
  verbose     = TRUE
)
```

Supported LLM providers:
- **Ollama** — local inference (e.g., `llama3.2`); preferred for privacy-sensitive clinical data
- **OpenAI** — cloud inference (e.g., `gpt-4o-mini`)
- **Anthropic** — cloud inference (e.g., `claude-3-5-sonnet-20241022`)

The system operates with BM25 keyword search alone if Ollama is unavailable. This BM25 fallback represents the [[concepts/fallback-strategy]] designed into the system to ensure baseline functionality regardless of local LLM availability.

### Semantic RAG for PAI Interpretation
The PAI pipeline queries a [[concepts/pai-knowledge-base]] built from up to 98 source documents chunked into approximately 4,830 retrievable segments:
- **Embedding model:** `snowflake-arctic-embed2:568m`
- **Search type:** Semantic (vector similarity)
- **Vector store:** [[concepts/duckdb-as-vector-store]] via `pai_knowledge_base_copy.duckdb`

### Core R Files

| File | Role |
|---|---|
| `rebuild_pai_ragnar.R` | Knowledge base construction from source PDFs |
| `pai_rag_system.R` | RAG search functions and LLM integration |
| `pai_score_interpreter.R` | PAI interpretation generator |
| `pai_complete_pipeline.R` | Interpretation generation and report formatting |
| `generate_as_interpretation.R` | Patient-specific report generation |
| `interpret_pai_from_scores.R` | Primary user-facing interface for score entry |
| `input/AS_scores_template.json` | Score entry template with patient demographics |
| `pai_knowledge_base.duckdb` | Persistent knowledge base (created by rebuild) |

See [[summaries/AS_PROCESSING_COMPLETE]], [[summaries/DEMO_GUIDE]], [[summaries/README_PIPELINE]], and [[summaries/README_WORKFLOW]] for documented demonstrations of this sub-pipeline.

---

## PHI & Privacy Constraints

Throughout all stages, the pipeline enforces strict anonymization. The neuropsych-pdf-parser is the **primary PHI scrubbing gate**: it replaces all patient identifiers with tokens (`[PATIENT]`, `[DOB]`, `[ID]`, `[CLINICIAN]`, `[FACILITY]`, `[CONTACT]`) before any text is passed downstream.

In the Streamlit app, this gate is implemented locally via Docling before any text is sent to Anthropic's API. No cloud vector store is used; LanceDB and SQLite are entirely local. MacWhisper transcription and oMLX summarization are local. The `.env` file is gitignored.

As a defense-in-depth measure, both the neuropsych-data-extractor and the narrative-writer include their own PHI checks: if real names or IDs slip through Stage 1 scrubbing, they replace them and add a WARNING.

For cloud LLM providers, users must weigh the privacy implications of transmitting T-scores and patient demographics. The local Ollama pathway (Luria) and local oMLX pathway (Streamlit app) are preferred in clinical environments with strict PHI obligations. See [[concepts/phi-data-handling]], [[concepts/pii-redaction-pipelines]], and [[concepts/privacy-first-software]].

## Quality Assurance

**Pre-generation checks:**
1. Run generic text-quality check (`check_pdf_text`) to flag NO_TEXT / POOR_TEXT PDFs.
2. For flagged PDFs, run OCR pre-processing (Stage 1a) before proceeding.
3. Run instrument-specific PDF assessment to determine correct extraction pathway.
4. Verify PDF data is complete and patient information is accurate.
5. Confirm test battery matches the assessment.
6. Validate lookup table entries.
7. For PAI: confirm all T-scores are sourced from actual bar graphs when graphical-only format is detected.
8. Double-check manual score entry — small errors can significantly change interpretations.
9. After any file reorganization or naming change, trigger a full knowledge base rebuild before processing new patients.
10. **Age group verification:** After ingestion, check for `age_group: "unknown"` entries. If found, add the report filename and correct age to `data/age_overrides.json` and re-run ingestion. See [[summaries/AGE_OVERRIDE_GUIDE]] for the procedure.

**Post-generation checks:**
1. Review PDF for formatting issues.
2. Verify all sections are included.
3. Validate diagnostic codes, signature block, and recommendations completeness.
4. For PAI: cross-check retrieved semantic contexts against clinical judgment before finalizing.
5. Validate OCR quality by comparing character counts before and after processing.
6. Review source chunk citations provided by the RAG system to verify accuracy.
7. For extractor CSV output: verify `ROW_COUNT`, `DOMAINS_DETECTED`, `RATERS`, `SCORE_TYPES`, and `WARNINGS`.
8. For narrative-writer output: review `FILES_WRITTEN`, `DOMAINS_SKIPPED`, and `EDIT_PROTECTION_HITS`.
9. For parser output: verify `DOCUMENT_TYPE` classification and review `PHI_FLAGS` log.
10. **Classification-vs-percentile consistency:** Verify that `range` label and percentile-derived clinical significance align. The NP-20240415-001 case demonstrated how a scaled score of 9 (37th percentile) can carry a normative label of "Average" while clinical thresholds flag it as below average — the narrative must resolve this explicitly.

## Structural Reorganization Risk

A recurring hazard in this codebase: structural reorganizations can cause old code to resurface and introduce subtle bugs. The renaming of `R/cingulate` to `engine/cingulate` under `~/luria` is one such change. Best practices after any reorganization:
- Audit all `source()` and `import` paths for stale references.
- Check that no legacy Google Sheets code paths have re-emerged.
- Verify Parquet/Arrow/CSV storage paths resolve correctly under the new folder structure.
- Trigger a full knowledge base rebuild before processing new patients.

## Clinical Disclaimers

All AI-generated interpretations in this pipeline are **AI-assisted, not replacements** for professional clinical judgment. All clinical decisions must be cross-checked with clinical interview and collateral information, other assessment data, and qualified professional review. The NP-20240415-001 case underscores this: a report generated from a single subtest requires explicit disclosure of its limited scope, and the pipeline's limitations section template should always flag single-test evaluations prominently.

## System Requirements

Full pipeline deployment requires:
- **Python** >=3.13 (extraction scripts, Streamlit app)
- **R** >=4.0 with neuro2, dplyr, ggplot2, knitr, tesseract, pdftools, fs, cli, duckdb, ragnar
- **Quarto** >=1.4.0
- **Typst** (latest)
- **Ollama** (local LLM backend; optional but recommended for Luria)
- **oMLX** (optional local OpenAI-compatible server for Streamlit app)
- **MacWhisper** CLI (`mw`) for audio transcription (Streamlit app, macOS)
- **OCRmyPDF** (for scanned PDF pre-processing; `brew install ocrmypdf`)
- **PyPDF2** (optional Python dependency for PDF text extraction via `extract_text()`)
- **uv** (recommended Python package manager)

Configuration is managed via [[concepts/yaml-configuration]] files (`config.yml`, `_variables.yml`) and environment variables (`.env`).

## Storage Architecture Summary

| Storage Layer | Technology | Purpose |
|---|---|---|
| Relational DB | SQLite | Structured test scores and clinical summaries |
| Columnar files | CSV / Arrow / Parquet | Primary data persistence via engine/cingulate |
| Vector Store (Streamlit) | LanceDB | Semantic retrieval of narrative chunks |
| Vector Store (PAI) | DuckDB | PAI knowledge base for semantic interpretation retrieval |
| Embedding Model (base) | all-MiniLM-L6-v2 | 384-dim text embeddings for RAG retrieval |
| Embedding Model (PAI) | snowflake-arctic-embed2:568m | Higher-capacity embeddings for PAI knowledge base |
| Age Override Config | JSON | Persistent age corrections for PDFs lacking extractable metadata |

## Relationship to Broader Agent Frameworks

The pipeline pattern mirrors multi-agent orchestration architectures described in [[concepts/multi-agent-orchestration]] and [[concepts/langgraph-agent-workflows]]. Each stage agent:
- Has a single, well-defined responsibility.
- Consumes a defined input schema.
- Produces a defined output schema.
- Fails loudly and explicitly rather than silently degrading.

This design makes the pipeline auditable, testable, and extensible — new stages can be inserted without disrupting existing stages.

---

## Related Pages

- [[summaries/AGE_OVERRIDE_GUIDE]] — Age override system for PDFs lacking extractable patient age
- [[summaries/text-extraction]] — `extract_text()` and `chunk_text()` module specification for Stage 1 file ingestion
- [[summaries/neuropsych-pdf-parser]] — Stage 1 parser agent: document classification, PHI scrubbing, and fixed-schema output
- [[summaries/neuropsych-data-extractor]] — Stage 2 extractor agent: long-format CSV schema, score-type mapping, and PHI rules
- [[summaries/neuropsych-narrative-writer]] — Stage 3 narrative-writer agent: per-domain prose, edit-protection, and output conventions
- [[summaries/OCR_PDF_GUIDE]] — OCR utilities and method selection for scanned PDF pre-processing
- [[summaries/AGENTS_luria]] — Specification of pipeline worker agents
- [[summaries/README]] — Luria Streamlit App: HIPAA-conscious local-first desktop implementation
- [[summaries/README_luria]] — Project-level context for the Luria system
- [[summaries/README_PIPELINE]] — PAI interpretation pipeline quick-start and file reference
- [[summaries/README_WORKFLOW]] — End-to-end PAI RAG workflow summary
- [[summaries/WORKFLOW_INSTRUCTIONS]] — Complete step-by-step operator guide for new patient processing
- [[summaries/responses_to_claude]] — User clarifications on plugin/subagent recommendations, engine renaming, and data format decisions
- [[summaries/report-generation]] — End-to-end workflow documentation
- [[summaries/report-rendering-pipeline]] — Step-by-step rendering pipeline: variable injection, format selection, Quarto render, Typst compile
- [[summaries/002-mcp-llm-integration]] — MCP and LLM integration details
- [[summaries/003-modular-template-structure]] — Modular template architecture
- [[summaries/template-system]] — Template system overview
- [[summaries/mcp-integration]] — MCP server integration
- [[summaries/overview]] — Project overview
- [[summaries/AS_PROCESSING_COMPLETE]] — PAI semantic RAG demonstration
- [[summaries/README_AS_PROCESSING]] — AS processing pipeline documentation
- [[summaries/DEMO_GUIDE]] — Demo workflow guide
- [[summaries/FIX_EXPLANATION]] — PDF assessment disconnect diagnosis and fix
- [[summaries/EMBEDDINGS_COMPLETE]] — Embeddings pipeline completion notes
- [[summaries/KNOWLEDGE_BASE_EXPLAINED]] — Knowledge base architecture explanation
- [[summaries/QUICK_REFERENCE]] — Quick reference for pipeline operations
- [[summaries/REBUILD_COMPLETE]] — Knowledge base rebuild completion notes
- [[summaries/REBUILD_FINAL_STATUS]] — Final rebuild status documentation
- [[summaries/SHINY_APP_FIXED]] — Shiny app fix notes
- [[summaries/TECHNICAL_DOCS]] — Technical documentation reference
- [[summaries/conversation-export]]
- [[summaries/processed_files]]
- [[summaries/SKILL]]
- [[summaries/clinical-validity-reviewer]]
- [[summaries/PROJECT_SETUP_COMPLETE]]
- [[summaries/RECOVERY_NOTES]]
- [[summaries/SESSION_SUMMARY_2025-04-28]]
- [[summaries/NP-20240415-001_report]] — Generated report for case NP-20240415-001 (WAIS-IV Digit Span only)
- [[summaries/report_body]] — AI agent thought process and output for NP-20240415-001 report generation
- [[concepts/age-group-classification]] — Age-to-group mapping governing norms and report register
- [[concepts/cingulate-engine]] — Core data processing engine replacing earlier Google Sheets workflows
- [[concepts/subagent-architecture]] — Design of parallel, dispatchable agents for pipeline stages
- [[concepts/ocr-pipeline]] — OCR methods and tooling for scanned document ingestion
- [[concepts/neuropsychological-tests]] — Instruments handled by the pipeline
- [[concepts/neuropsychological-test-scores]] — Score types extracted and normalized
- [[concepts/neuropsychological-reporting]] — Output conventions for the final report stage
- [[concepts/cognitive-domains]] — Domain taxonomy used in analysis
- [[concepts/phi-data-handling]] — Privacy constraints enforced across all stages
- [[concepts/pii-redaction-pipelines]] — Anonymization mechanisms
- [[concepts/privacy-first-software]] — Local-first design philosophy supporting PHI compliance
- [[concepts/retrieval-augmented-generation]] — RAG techniques applied via the vector store index
- [[concepts/multi-agent-orchestration]] — Architectural pattern underpinning the pipeline
- [[concepts/langgraph-agent-workflows]] — Workflow orchestration technology applicable to this pipeline
- [[concepts/clinical-nlp-pipelines]] — NLP techniques applied in analysis stages
- [[concepts/clinical-pdf-assessment]] — PDF assessment logic distinguishing text vs. graphical score formats
- [[concepts/pdf-score-extraction]] — Score extraction methods and format detection
- [[concepts/local-llm-inference]] — Local LLM backend for PDF extraction
- [[concepts/model-context-protocol]] — MCP server integration for LLM invocation
- [[concepts/long-format-clinical-data]] — Long-format data schema used by cingulate
- [[concepts/narrative-report-generation]] — Narrative prose generation for clinical reports
- [[concepts/edit-protection-pattern]] — Pattern for protecting clinician hand-edits from automated overwrites
- [[concepts/text-chunking]] — Text chunking strategy for embedding preparation
- [[concepts/quarto]] — Document generation framework used in rendering
- [[concepts/quarto-extensions]] — Format-specific Quarto extensions
- [[concepts/typst-typesetting]] — Final PDF compilation engine
- [[concepts/r-python-integration]] — R and Python tooling across pipeline stages
- [[concepts/yaml-configuration]] — Configuration management across all stages
- [[concepts/pai-assessment]] — PAI instrument and its specialized pipeline extension
- [[concepts/pai-knowledge-base]] — 98-document corpus for semantic PAI interpretation
- [[concepts/duckdb-as-vector-store]] — DuckDB as the vector store backend for PAI knowledge base
- [[concepts/vector-search]] — Vector similarity search underpinning semantic retrieval
- [[concepts/fallback-strategy]] — BM25 fallback when Ollama/semantic search is unavailable
- [[concepts/r-visualization-theming]] — R visualization tooling used in data processing stage
- [[concepts/pass-theory]] — Planning, Attention, Simultaneous, Successive cognitive model tagged in score rows
- [[concepts/modular-report-architecture]] — Modular include-file pattern used by the narrative-writer
- [[concepts/lancedb-vector-store]] — LanceDB vector store used in the Streamlit app implementation
- [[concepts/docling-pdf-parsing]] — Docling PDF parser used in the Streamlit app for local extraction
- [[concepts/mild-cognitive-impairment]] — Diagnostic category illustrated by the NP-20240415-001 case
- [[concepts/working-memory]] — Cognitive construct measured by the WAIS-IV Digit Span in NP-20240415-001

See also: [[summaries/multi_patient_transcript]]

See also: [[summaries/0010-voice-quarto-typst-reporting]]

See also: [[summaries/report-generator]]

See also: [[summaries/full-pipeline]]

See also: [[summaries/agent-team]]

See also: [[summaries/2026-04-26-cingulate-agent-team-design]]

See also: [[summaries/CLAUDE]]

See also: [[summaries/extract_pdf]]

See also: [[summaries/DIAGNOSIS_FIX_SUMMARY]]

See also: [[summaries/DIAGNOSIS_PARSER_IMPROVEMENTS]]

See also: [[summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_CODEMAP]]

See also: [[summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_PIPELINE]]

See also: [[summaries/PERMANENT_SOLUTION_SUMMARY]]