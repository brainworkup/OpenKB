---
sources: [summaries/full-pipeline.md, summaries/customization.md, summaries/style-training-to-report-drafting.md, summaries/style-trainer.md, summaries/soul-style-agent.md, summaries/report-generator.md, summaries/0008-soul-single-file-style-agent-architecture.md, summaries/0004-soul-style-profile-json.md, summaries/README.md, summaries/neuropsych-narrative-writer.md, summaries/clinical-validity-reviewer.md, summaries/index.md]
brief: Extracting a reusable JSON style profile from a report corpus to constrain LLM-generated clinical text.
---

# Style Profile Extraction

Style profile extraction is the process of analyzing a corpus of example documents and distilling their voice, tone, structural patterns, and linguistic habits into a structured, reusable JSON profile that can be injected into LLM prompts to constrain generated output.

## Core Idea

Rather than relying on ad-hoc instructions each time an LLM generates text, style profile extraction creates a **persistent, data-driven description** of how a body of writing sounds and is organized. This profile becomes a first-class artifact — stored, versioned, and reloaded — so that every generation pass stays consistent with the source material's identity.

This approach is especially valuable in high-stakes domains like [[concepts/neuropsychological-reporting]], where tone, register, and structure must remain clinically appropriate and consistent across many reports written by different practitioners.

## Reference Implementation

The canonical implementation is the `neuro_report_style_agent.py` single-file agent (see [[summaries/soul-style-agent]] and [[summaries/0008-soul-single-file-style-agent-architecture]]). It follows a [[concepts/single-file-agent-pattern]] approach — all logic lives in one Python file with stdlib-only runtime dependencies (`urllib`, `sqlite3`, `json`, `argparse`), making the style extraction pipeline fully self-contained and local-first.

The `train-style` subcommand (documented in [[summaries/style-trainer]] and [[summaries/0004-soul-style-profile-json]]) implements style profile extraction in four steps:

1. **Seed RAG Query** — A fixed semantic query (*"Identify writing style, narrative voice, phrasing patterns, section structure, and professional tone in neuropsychological reports."*) is embedded and used to retrieve the most representative chunks (default top-12) from the indexed corpus via [[concepts/retrieval-augmented-generation]].
2. **Corpus Assembly** — Exemplar chunks are formatted with source attribution (`[Source: path#chunk_id]`) to preserve traceability of which documents influenced the final profile.
3. **LLM Call** — An LLM call via the [[concepts/omlx-server]] (see [[concepts/local-llm-inference]] and [[concepts/fallback-strategy]]) produces the profile JSON at low temperature (0.1) to minimize variance and ensure reproducibility.
4. **Post-Processing** — The output is validated for required keys and written to `report_style_profile.json`.

### CLI Invocation

```bash
python neuro_report_style_agent.py train-style \
    --db-path ./report_style_index.sqlite \
    --profile-path ./report_style_profile.json \
    --style-examples 12
```

### Parameters

| Flag | Default | Description |
| ---- | ------- | ----------- |
| `--db-path` | required | Path to indexed SQLite database |
| `--profile-path` | required | Output path for JSON profile |
| `--style-examples` | 12 | Number of exemplar chunks to analyze |

## The Style Profile Schema

The extracted profile is stored as a JSON document with the following fixed schema (as defined in ADR 0004):

```json
{
  "voice": "Professional clinical analyst...",
  "tone": "Objective yet empathetic...",
  "structure_patterns": [
    "Opening with patient demographic context",
    "Presenting findings in order of clinical significance",
    "Closing with actionable recommendations"
  ],
  "typical_phrases": [
    "Performance on [test] was within expected range",
    "Notably, [observation]",
    "These findings suggest..."
  ],
  "do_rules": [
    "Use person-first language",
    "Include specific test scores with percentiles",
    "Distinguish between performance and capacity"
  ],
  "avoid_rules": [
    "Avoid definitive diagnostic statements without context",
    "Do not use casual contractions",
    "Never speculate beyond the data"
  ]
}
```

This schema maps directly to prompt-injectable text. See [[concepts/clinical-communication-register]] for why these distinctions matter in clinical contexts.

All fields are manually editable. Clinicians or administrators can open the JSON file directly and adjust `voice`, `tone`, `structure_patterns`, `do_rules`, and `avoid_rules` without re-running training — useful for rapid iteration on specific behavioral constraints.

## Usage in Generation Prompts

The profile is embedded directly into generation prompts using Python's `json.dumps()`, making the style profile a first-class prompt engineering artifact:

```python
report_prompt = f"""
STYLE PROFILE (must follow):
{json.dumps(profile, indent=2)}

USER TASK:
{args.prompt}
...
"""
```

During `write-report`, both the style profile **and** live-retrieved context chunks are included in the prompt, creating a two-layer grounding mechanism. The profile governs *how* to write; the retrieved chunks provide *what* to write about. The generation prompt also encodes safety rules — the LLM is explicitly instructed not to fabricate patient facts and to mark any missing data with `[NEEDS DATA]` rather than inventing details.

### `write-report` CLI Invocation

```bash
python neuro_report_style_agent.py write-report \
    --db-path ./report_style_index.sqlite \
    --profile-path ./report_style_profile.json \
    --prompt "Write a concise summary for a 12-year-old with attention concerns." \
    --top-k 6 \
    --temperature 0.2 \
    --output ./draft_report.txt
```

## Role in the Three-Stage Pipeline

Style profile extraction is the second of three stages in the full report generation pipeline. The complete workflow — documented in [[summaries/full-pipeline]] and [[summaries/style-training-to-report-drafting]] — chains all three stages:

```
1. build-index  → index historical PDF/TXT/MD reports into SQLite
2. train-style  → extract style profile from indexed chunks
3. write-report → generate new draft using RAG + style profile
```

A one-shot bash script automates the full sequence with `set -e` for fail-fast behavior:

```bash
#!/bin/bash
set -e

REPORTS_DIR="./neuropsych_report_pdf_bucket"
DB_PATH="./report_style_index.sqlite"
PROFILE_PATH="./report_style_profile.json"
OUTPUT_PATH="./draft_report.txt"

python soul/neuro_report_style_agent.py build-index \
    --reports-dir "$REPORTS_DIR" --db-path "$DB_PATH"

python soul/neuro_report_style_agent.py train-style \
    --db-path "$DB_PATH" --profile-path "$PROFILE_PATH"

python soul/neuro_report_style_agent.py write-report \
    --db-path "$DB_PATH" --profile-path "$PROFILE_PATH" \
    --prompt "Write a comprehensive summary section for an 8-year-old with ASD and average cognitive abilities." \
    --output "$OUTPUT_PATH"
```

The first stage (`build-index`) uses fixed-size overlapping chunking (default 1200 chars, 150 overlap), embeds chunks via the [[concepts/omlx-server]] `/embeddings` endpoint, and stores them in a [[concepts/sqlite-as-vector-store]] `chunks` table. Supported input formats are `.pdf`, `.txt`, and `.md`. At generation time (`write-report`), the profile is loaded from disk, the user prompt is embedded, and top-k relevant chunks are retrieved using pure-Python cosine similarity over the SQLite index. Temperature is kept low (default 0.2) to favor consistency over creative variation.

The draft output (`draft_report.txt` or `.qmd`) is manually integrated into Quarto Markdown section files and rendered via [[concepts/quarto]] to produce a final PDF.

## Incremental Updates

When new reports are added to the corpus:

```bash
# Re-run build-index (incremental via INSERT OR REPLACE)
python soul/neuro_report_style_agent.py build-index \
    --reports-dir ./neuropsych_report_pdf_bucket \
    --db-path ./report_style_index.sqlite

# Optionally retrain style profile
python soul/neuro_report_style_agent.py train-style \
    --db-path ./report_style_index.sqlite \
    --profile-path ./report_style_profile.json
```

Style profiles are **not auto-updated**; regeneration requires explicitly re-running `build-index` then `train-style` after adding new exemplar documents.

## Iterative Drafting

Multiple versions can be generated at different temperatures to compare conservative vs. creative output:

```bash
# Conservative draft
python ... write-report ... --temperature 0.1 --output draft_v1.txt

# Balanced draft
python ... write-report ... --temperature 0.2 --output draft_v2.txt

# Creative draft
python ... write-report ... --temperature 0.3 --output draft_v3.txt
```

## Customizing Style Profiles

The style profile system supports several dimensions of customization, documented in [[summaries/customization]].

### Per-Clinician Profiles

Individual clinicians can have dedicated profiles trained exclusively from their own historical reports:

```bash
# Train profile from Dr. Smith's reports only
python soul/neuro_report_style_agent.py build-index \
    --reports-dir ./reports/smith/ \
    --db-path ./smith_index.sqlite

python soul/neuro_report_style_agent.py train-style \
    --db-path ./smith_index.sqlite \
    --profile-path ./smith_profile.json
```

### Per-Population Profiles

Separate profiles can be maintained for distinct patient populations:
- **Pediatric** — developmental focus, age-appropriate language, developmental history context.
- **Adult** — standard neuropsychological framing.
- **Forensic** — heightened objectivity and precision requirements (see [[concepts/forensic-neuropsychological-evaluation]]).

### A/B Profile Testing

Two profiles can be compared by generating parallel drafts and running `diff` to evaluate stylistic differences:

```bash
python ... write-report --profile-path ./profile_a.json --output draft_a.txt
python ... write-report --profile-path ./profile_b.json --output draft_b.txt
diff draft_a.txt draft_b.txt > comparison.txt
```

## Custom Chunking Strategies

Chunk size affects how reports are indexed and retrieved, connecting to the [[concepts/rag-chunking]] and [[concepts/text-chunking]] strategies:

| Use Case | `--chunk-size` | `--overlap` |
|---|---|---|
| Long-form comprehensive reports | 2000 | 200 |
| Brief screening reports | 600 | 75 |
| Default | ~1200 | ~150 |

Section-aware chunking can be achieved by pre-extracting named sections before indexing, attaching section metadata for more targeted retrieval.

## Custom Retrieval Parameters

Retrieval behavior is tuned via `--top-k` (number of retrieved chunks) and `--temperature` (generation randomness):

| Mode | `--top-k` | `--temperature` |
|---|---|---|
| High-context (complex cases) | 12 | 0.15 |
| Quick drafting (standard cases) | 4 | 0.25 |
| Balanced (default) | 6 | 0.2 |
| Style extraction (train-style) | 12 | 0.1 |

## Custom Model Configuration

- **Embedding model**: Override with `--omlx-embed-model` (e.g., `nomic-embed-text-v1.5`; default is `nomicai-modernbert-embed-base-bf16`).
- **Generation model**: Override with `--omlx-gen-model` (e.g., `llama-3.1-70b`; default is `Qwopus3.5-9B-v3-PolarQuant-MLX-4bit`).
- **Alternative backends**: Ollama can be substituted by setting `OMLX_URL` to the Ollama endpoint (`http://localhost:11434/v1`).

This flexibility is supported by the [[concepts/openai-compatible-api]] contract shared by OMLX, Ollama, and other local inference servers. See [[concepts/llm-provider-abstraction]] for the broader pattern of swappable backends.

## Custom Report Sections and Output Formats

Prompt engineering enables generation of targeted sections — executive summaries, background/history sections, test results tables, and domain-organized recommendations. Each section can be generated independently and concatenated into a full report via shell scripting, allowing modular review and editing.

Output formats include:
- **Markdown**: Default `--output report.md`.
- **Structured JSON**: Programmatic extension to emit sections and metadata as a JSON object, useful for downstream processing or integration with external systems.

## Integration with Quarto Rendering

Once `write-report` produces a draft, the text is copied into the appropriate `.qmd` section file (e.g., `_01-01_behav_obs.qmd`) and included in the main template via `{{< include >}}`. The full rendering step is:

```bash
quarto render style/templates/typst-report/template.qmd
```

This positions style profile extraction as a bridge between the AI drafting layer and the structured document rendering layer. See [[summaries/report-rendering-pipeline]] for how the rendering side is organized, and [[concepts/typst-typesetting]] for the underlying typesetting engine.

## Configuration and Backend Overrides

All pipeline CLI flags are overridable, allowing the same workflow to run against alternative inference backends:

```bash
python neuro_report_style_agent.py write-report \
    --omlx-url http://localhost:11434/v1 \
    --omlx-embed-model nomic-embed-text \
    --omlx-gen-model llama3.1 \
    --top-k 10 \
    --temperature 0.3 \
    --db-path ./report_style_index.sqlite \
    --profile-path ./report_style_profile.json \
    --prompt "..." \
    --output ./draft.txt
```

## Relationship to RAG

Style profile extraction is a specialized use of [[concepts/retrieval-augmented-generation]]: instead of retrieving context to answer a factual question, retrieval is used to surface the most representative examples of a writing style. The LLM then performs a *meta-synthesis* — summarizing not facts, but patterns.

The extracted profile complements but does not replace RAG at generation time. Both mechanisms operate together: the profile constrains style, while live retrieval grounds content.

## Exemplar Retrieval Design

A key design decision is the use of a **fixed seed query** for exemplar retrieval. This guarantees consistent chunk retrieval across runs regardless of corpus changes, making the process reproducible. The `embed_with_fallback` / `generate_with_fallback` pattern (see [[concepts/fallback-strategy]]) provides single call-sites for all embedding and generation operations. Cosine similarity is computed in pure Python without external numeric libraries, keeping the runtime dependency footprint minimal.

**Source attribution in corpus assembly** preserves traceability: each chunk is prefixed with `[Source: path#chunk_id]`, so the influence of any given historical report on the resulting profile can be traced.

## Storage and Persistence

The profile is stored as a plain JSON file (`report_style_profile.json`), with several practical advantages confirmed in ADR 0004:

- **Human Readable** — Clinicians can review and manually edit profiles without special tooling
- **Version Control Friendly** — JSON diffs cleanly in git
- **Language Agnostic** — Any tool or language can consume the file
- **Extensible** — New fields can be added without breaking existing consumers
- **Decoupled** — Independent from the vector index ([[concepts/sqlite-as-vector-store]])
- **Reloadable** — Persists across sessions without re-running training

This aligns with the plain-text documentation philosophy: artifacts that humans can read and audit are preferred over opaque binary formats.

## Profile Versioning

Profile versioning is recommended to support A/B testing across clinician styles, population types, or time periods:

```bash
report_style_profile.v1.json
report_style_profile.v2.json
smith_profile.json
pediatric_profile.json
forensic_profile.json
```

This allows comparison of different style periods, different clinician preferences, or the effect of corpus expansion.

### Iteration Workflow

1. Add new reports to the PDF bucket
2. Re-run `build-index` (incremental via `INSERT OR REPLACE`)
3. Re-run `train-style` to generate updated profile

## Performance Optimization

For large corpora (1000+ reports), batch indexing by year followed by database merging is recommended:

```bash
for year in 2020 2021 2022 2023 2024; do
    python ... build-index --reports-dir ./reports/$year --db-path ./index_${year}.sqlite
done
python merge_indices.py --inputs ./index_*.sqlite --output ./master_index.sqlite
python ... train-style --db-path ./master_index.sqlite ...
```

Parallel PDF extraction using GNU `parallel` can further accelerate ingestion:

```bash
find ./reports -name "*.pdf" | parallel python extract_and_index.py {}
```

A minimum corpus of **20–30 reports** is recommended for meaningful style extraction.

## Quality Assurance Workflow

Post-generation automated checks help catch common issues before clinician review:

```bash
# Check for unfilled placeholders
grep -n "NEEDS DATA" draft.txt || echo "None found - good"

# Check for prohibited casual contractions
grep -in "don't\|can't\|won't\|isn't" draft.txt || echo "None found - good"

# Word count validation
wc -w draft.txt
```

Post-generation review remains mandatory: clinicians must verify scores, check patient identifiers, and ensure recommendations are appropriate before any draft is finalized. See [[concepts/report-review-qa]] for quality assurance considerations.

### Quality Checklist

Before integrating a generated draft into the report pipeline:

- [ ] Profile `voice` matches clinician's first-person narrative style
- [ ] `do_rules` enforce plain language and active voice
- [ ] `avoid_rules` prevent passive voice and jargon
- [ ] `typical_phrases` include domain-specific terminology
- [ ] Draft output is reviewed by clinician before insertion into `.qmd`
- [ ] `[NEEDS DATA]` placeholders are replaced with actual patient data

## Error Handling

- **No indexed chunks**: `RuntimeError("No indexed chunks found. Run build-index first.")` — style training requires the index to be built first
- **Invalid JSON from LLM**: `json.JSONDecodeError` propagates to the user — low temperature (0.1) reduces but does not eliminate this risk; retry with `--style-examples 8` if needed
- **Missing DB file**: SQLite connection error on attempt
- **Missing PDF dependency**: `"PyPDF2 is required"` — resolved with `pip install PyPDF2`
- **Connection errors**: OMLX not running — start OMLX server on port 8000

After extraction, the implementation validates that all required keys are present to guard against LLM outputs that omit required fields or produce malformed JSON.

## Safety Guardrails

The style profile works in concert with built-in generation rules to ensure clinical safety:

- **No fabrication** — The generation prompt explicitly prohibits inventing patient data or test scores
- **`[NEEDS DATA]` placeholder** — Missing information is flagged rather than fabricated
- **Style adherence** — The loaded profile rules constrain register and structure
- **Draft framing** — Output is always framed as a draft requiring clinician review and sign-off

## Trade-offs and Limitations

ADR 0004 identifies several known limitations:

- **No Validation** — Schema enforcement is informal; no enforced contract prevents missing or malformed fields
- **LLM Dependency** — Profile quality is only as good as the LLM's analysis of the source corpus
- **Static** — Profiles do not self-update; they require explicitly re-running `train-style` to incorporate new exemplar documents
- **Generic output risk** — Poor retrieval (low `--top-k` or sparse corpus) can produce non-specific drafts

## Extensibility

- **Custom exemplar count** — A `--style-examples` flag controls how many chunks seed the extraction (default 12)
- **Domain adaptation** — The schema can be extended with domain-specific keys (e.g., `referral_language`, `score_reporting_style`) for different clinical specialties
- **Profile versioning** — Multiple profiles can be trained on different sub-corpora and selected at generation time
- **Parameter tuning** — Generation parameters such as `--top-k` and `--temperature` can be tuned independently of the profile itself
- **Backend flexibility** — The `--omlx-url`, `--omlx-embed-model`, and `--omlx-gen-model` flags decouple the pipeline from any specific inference server

## Related Concepts

- [[concepts/retrieval-augmented-generation]] — Powers both the chunk retrieval step during extraction and grounding at generation time
- [[concepts/neuropsychological-reporting]] — Primary domain of application
- [[concepts/clinical-communication-register]] — Defines what "appropriate tone" means clinically
- [[concepts/sqlite-as-vector-store]] — Storage layer for the indexed chunks
- [[concepts/local-llm-inference]] — Inference backend used for both extraction and generation
- [[concepts/omlx-server]] — The specific local inference server used at `http://127.0.0.1:8000/v1`
- [[concepts/fallback-strategy]] — Ensures extraction and generation work even if the primary backend is unavailable
- [[concepts/single-file-agent-pattern]] — Architectural pattern used by the reference implementation
- [[concepts/openai-compatible-api]] — API contract fulfilled by the OMLX backend
- [[concepts/persistent-memory]] — Broader context for retaining learned agent behaviors across sessions
- [[concepts/narrative-report-generation]] — Output artifact produced using the style profile
- [[concepts/llm-provider-abstraction]] — Pattern for swapping inference backends without changing pipeline logic
- [[concepts/quarto]] — Rendering system that consumes the drafted `.qmd` section files
- [[concepts/typst-typesetting]] — Underlying typesetting engine for PDF output
- [[concepts/report-review-qa]] — Quality assurance layer downstream of style-constrained generation
- [[concepts/rag-chunking]] — Chunking strategies that affect what exemplars are retrieved during extraction
- [[concepts/text-chunking]] — Lower-level chunking mechanics used during `build-index`
- [[concepts/forensic-neuropsychological-evaluation]] — One of the population-specific profile types supported

## Source Documents

- [[summaries/soul-style-agent]] — Full agent architecture overview including all three pipeline stages
- [[summaries/0004-soul-style-profile-json]] — ADR defining the JSON schema, generation process, and trade-offs
- [[summaries/0008-soul-single-file-style-agent-architecture]] — Architecture of the agent module containing both `train-style` and `write-report`
- [[summaries/0009-soul-local-llm-inference-with-omlx]] — OMLX inference backend used during extraction and generation
- [[summaries/style-trainer]] — Detailed documentation of the `train-style` subcommand, including workflow, prompt, schema, and CLI parameters
- [[summaries/style-training-to-report-drafting]] — End-to-end workflow connecting style training to Quarto PDF rendering
- [[summaries/full-pipeline]] — Complete three-stage pipeline with one-shot bash script and incremental update instructions
- [[summaries/report-generator]] — Full specification of the `write-report` command that consumes the profile
- [[summaries/report-generation]] — Related report generation pipeline overview
- [[summaries/report-rendering-pipeline]] — How drafted sections are rendered into final PDF reports
- [[summaries/clinical-validity-reviewer]] — Downstream consumer of style-constrained reports
- [[summaries/neuropsych-narrative-writer]] — Narrative generation component that uses style profiles
- [[summaries/customization]] — Detailed guidance for per-clinician, per-population, and parameter-level customization
- [[summaries/README]] — Project overview