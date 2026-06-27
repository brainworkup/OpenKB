---
doc_type: short
full_text: sources/report-generator.md
---

# Report Generator Component

## Overview

The Report Generator is the final stage in a three-part pipeline for producing neuropsychological report drafts. It uses [[concepts/retrieval-augmented-generation]] (RAG) to combine a user-supplied prompt, a clinician style profile, and historically indexed report chunks to produce a polished draft section suitable for clinician review.

**Module**: `soul/neuro_report_style_agent.py:282-319`  
**Command**: `write-report`

---

## Workflow

1. Load the style profile from JSON (see [[concepts/style-profile-extraction]])
2. Embed the user prompt (via `embed_with_fallback`)
3. Retrieve top-k relevant chunks from [[concepts/sqlite-as-vector-store]] index (via `retrieve_top_k`)
4. Assemble a generation prompt combining style + retrieved context
5. Call LLM via `generate_with_fallback` at a specified temperature (see [[concepts/fallback-strategy]])
6. Output draft to file or stdout

---

## Key Parameters

| Parameter | Flag | Default |
|-----------|------|---------|
| Database path | `--db-path` | — |
| Style profile | `--profile-path` | — |
| User prompt | `--prompt` | — |
| Top-k chunks | `--top-k` | 6 |
| Temperature | `--temperature` | 0.2 |
| Output file | `--output` | stdout |

---

## Retrieval & Context Assembly

- The user prompt is embedded and used to query the SQLite-backed report chunk index.
- Retrieved chunks are formatted with source attribution (`[Reference: path#chunk_id]`) before being injected into the generation prompt.
- Default retrieval is top-6 chunks; this can be increased (e.g., `--top-k 8`) for broader context.
- This retrieval approach connects to the broader [[concepts/vector-search]] and [[concepts/hybrid-search-retrieval]] strategies used across the system.

---

## Generation Prompt Structure

The assembled prompt passed to the LLM includes three sections:
1. **Style Profile** — JSON rules the output must follow (see [[concepts/style-profile-extraction]])
2. **User Task** — The specific drafting instruction
3. **Retrieval Context** — Historical chunks for content grounding

Key rules embedded in the prompt:
- Do not fabricate patient facts
- Mark missing data as `[NEEDS DATA]`
- Keep clinical language professional (see [[concepts/clinical-communication-register]])
- Frame output as a draft for clinician review

**Temperature**: 0.2 (low, favoring consistency over creativity)

---

## Safety Guardrails

- **No hallucination**: Explicit instruction to avoid inventing test scores or patient data
- **`[NEEDS DATA]` placeholder**: Forces acknowledgment of missing information rather than fabrication
- **Style adherence**: Output constrained by the loaded profile
- **Post-generation review required**: Clinician must verify scores, identifiers, recommendations, and add signature/disclaimer (see [[concepts/report-review-qa]])

---

## Prompt Engineering Guidance

Effective prompts should:
- Be specific about section type (summary, recommendations, test interpretation)
- Include patient demographics (age, presenting concerns)
- Specify constraints (length, tone, required elements)

### Example Prompts
- `"Write a 2-paragraph summary for a 9-year-old with above-average cognitive abilities but weaknesses in processing speed."`
- `"Interpret WISC-V results for a teenager: VCI 115, PRI 108, WMI 92, PSI 85."` (see [[concepts/neuropsychological-test-scores]])
- `"Generate 5 specific school recommendations for a child with executive function deficits."` (see [[concepts/executive-function-deficits]])

---

## Pipeline Position

This component is **Step 3** of the full pipeline:

```
1. build-index  → indexes historical PDF reports into SQLite
2. train-style  → extracts style profile from the index
3. write-report → generates new draft using RAG
```

See also: [[concepts/neuropsychological-assessment-pipeline]], [[concepts/narrative-report-generation]], [[concepts/retrieval-augmented-generation]], [[concepts/single-file-agent-pattern]]

## Related Concepts
- [[concepts/llm-provider-abstraction]]
- [[concepts/clinical-data-privacy]]
- [[concepts/clinical-report-structure]]
