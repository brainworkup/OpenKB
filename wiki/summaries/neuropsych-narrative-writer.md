---
doc_type: short
full_text: sources/neuropsych-narrative-writer.md
---

# neuropsych-narrative-writer

## Overview

`neuropsych-narrative-writer` is **stage 3 of the Luria neuropsychological assessment pipeline**. Its sole responsibility is generating **per-domain prose narratives** as Quarto include files (`.qmd`) that drop into the [[concepts/quarto]] and [[concepts/typst-typesetting]] pipeline for final PDF rendering. It does not produce Word documents, Google Docs, PDFs, or any standalone report format.

## Role in the Pipeline

- **Stage 1–2**: Data extraction (CSV output).
- **Stage 3** (this agent): Narrative writing — prose only, no tables or plots.
- **R6 layer / cingulate**: Renders score tables, plots, and assembles the Typst PDF.

See also: [[concepts/luria-neuropsych-pipeline]], [[concepts/neuropsychological-assessment-pipeline]], and [[concepts/cingulate-engine]].

## Output Files

For each cognitive domain present in the extracted CSV, the agent writes a file named:

```
_NN-XX_<domain>_text.qmd
```

Domain prefix table:

| Prefix | Domain |
|---|---|
| `_02-01_iq` | General Cognitive Ability / IQ |
| `_02-02_academics` | Academic / Achievement |
| `_02-03_verbal` | Verbal / Language |
| `_02-04_spatial` | Visuospatial / Visual-Construction |
| `_02-05_memory` | Memory & Learning |
| `_02-06_executive` | Executive Function |
| `_02-07_motor` | Sensorimotor |
| `_02-08_social` | Social Cognition |
| `_02-09_adhd` | ADHD (multi-rater: `_self`, `_parent`, `_teacher`) |
| `_02-10_emotion` | Emotion/Behavior (multi-rater) |
| `_02-11_adaptive` | Adaptive Functioning |
| `_03-00_sirf` | Summary, Impressions, Recommendations & Formulation |
| `_03-01_recs` | Recommendations |

Prefixes must **never be renumbered** — `_quarto.yml` and `template.qmd` depend on the stable ordering. This maps directly to [[concepts/modular-report-architecture]] and [[concepts/cognitive-domains]].

## Workflow Steps

1. Read the extractor's CSV (via `Read` or `Glob`).
2. For each subdomain, draft **2–4 short paragraphs** covering:
   - **Performance summary** — what was tested; qualitative range drawn verbatim from the `range` column.
   - **Pattern interpretation** — relative strengths/weaknesses, intra-test scatter, score-type discrepancies.
   - **Functional implication** — one hedged sentence linking to everyday/academic/occupational impact.
3. Write each domain file to the patient workspace.
4. **Multi-rater domains** (ADHD, Emotion/Behavior): produce one file per rater present in the data; skip missing raters entirely.
5. **Edit-protection check**: read any existing `_text.qmd` before overwriting. If clinician hand-edits are detected, append the new draft as a `<!-- DRAFT: ... -->` comment and report a warning. See [[concepts/edit-protection-pattern]].

## Voice and Style

- Professional, APA-style neuropsychological report register consistent with [[concepts/clinical-communication-register]].
- Hedged language: *"performance is consistent with…"*, *"results suggest…"*, *"indicates relative weakness in…"* — see [[concepts/validity-language]].
- Adapts register to `age_group` column: pediatric, adult, or forensic (see [[concepts/forensic-neuropsychological-evaluation]]).
- Can mirror a `STYLE_PROFILE` or `EXEMPLAR_SNIPPETS` block if provided — related to [[concepts/style-profile-extraction]] — but **only the CSV is the evidence base**.
- Markdown only — no raw HTML; Quarto-compatible.

## Hard Rules

- **No raw scores or percentiles in prose** — the R6 layer handles score tables. See [[concepts/neuropsychological-test-scores]].
- **No diagnoses, etiology, or prognosis** unless explicitly present in the source extraction.
- **No output outside the patient workspace** — never write to cingulate's package internals or R source files.
- **PHI handling**: assumes upstream scrubbing; if real names appear, replace with `[PATIENT]` and emit a WARNING. See [[concepts/phi-data-handling]] and [[concepts/pii-redaction-pipelines]].
- Output format is exclusively Quarto (`.qmd`) for the [[concepts/quarto-extensions]] and Typst rendering pipeline.

## Final Message Format

The agent concludes each run with a structured summary:

```
WORKSPACE_DIR: <path>
FILES_WRITTEN: <list with domain and paragraph count>
DOMAINS_SKIPPED: <subdomains with insufficient data>
EDIT_PROTECTION_HITS: <files where overwrite was blocked>
NEXT_STEP: quarto render or cingulate_workflow()
```

## Related Pages

- [[concepts/luria-neuropsych-pipeline]] — the multi-stage pipeline this agent belongs to
- [[concepts/neuropsychological-reporting]] — broader reporting context
- [[concepts/narrative-report-generation]] — narrative prose generation patterns
- [[concepts/modular-report-architecture]] — the domain-numbered file structure
- [[concepts/cognitive-domains]] — the domain taxonomy this agent operates over
- [[concepts/clinical-report-structure]] — overall report organization
- [[summaries/neuropsych-data-extractor]] — the upstream stage that produces the CSV input
- [[concepts/long-format-clinical-data]] — the data format consumed by this agent

## Related Concepts
- [[concepts/clinical-nlp-pipelines]]
- [[concepts/multi-agent-orchestration]]
