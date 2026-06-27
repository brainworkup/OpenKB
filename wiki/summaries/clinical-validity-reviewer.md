---
doc_type: short
full_text: sources/clinical-validity-reviewer.md
---

# Clinical Validity Reviewer

## Overview

The `clinical-validity-reviewer` is a read-only parallel agent in Luria's neuropsychological report pipeline. It reviews draft narrative files (`_text.qmd`) and extracted score data (CSV) before sign-out, producing a structured punch list of issues without modifying any files. It is designed to run in parallel with the narrative-writer agent (see [[concepts/narrative-report-generation]]).

## Inputs

- **`csv_path`** — Long-format CSV output from the score extractor ([[concepts/neuropsychological-assessment-pipeline]] cingulate schema)
- **`qmd_dir`** — Directory of draft domain narrative files (`_NN-XX_<domain>_text.qmd`)
- **Optional:** `referral_question`, `validity_test_battery`, `style_profile`

## Six Review Axes

### 1. Completeness
- Every CSV subdomain has a corresponding narrative file or a documented skip.
- Multi-rater domains (ADHD, emotion) cover all raters present in the data.
- The narrative addresses the referral question if one was supplied.

### 2. Validity Language
- If validity/effort tests are present, a validity statement must appear in the narrative.
- Suspect-effort findings must be hedged appropriately (e.g., "performance suggests insufficient effort").
- Prohibits definitive language like "malingering" or "feigned" without positive validity findings.
- Prohibits overcertain language ("conclusively", "proves") on equivocal findings.
- See [[concepts/validity-language]] for hedging standards.

### 3. Premorbid Context
- Narrative must integrate premorbid functioning (education, occupation, baseline) where available.
- Discrepancies between current performance and premorbid expectations must be noted.
- Cultural, linguistic, and sensorimotor confounds must be flagged when relevant.

### 4. Score–Narrative Consistency
- Every qualitative claim must be backed by a CSV row.
- Score ranges in the narrative must match the CSV `range` column exactly.
- No invented diagnoses or test names — cross-checked against CSV `test`/`test_name` fields.
- Related to [[concepts/neuropsychological-test-scores]] and [[concepts/neuropsychological-reporting]].

### 5. PHI Leak Detection
- Grep-based scan for: real names, DOB patterns, SSNs, MRNs (≥6 digits), phone numbers.
- Flags literal addresses, emails, and facility names.
- Requires use of placeholder tokens: `[PATIENT]`, `[CLINICIAN]`, `[FACILITY]`.
- See [[concepts/phi-data-handling]], [[concepts/pii-redaction-pipelines]], and [[concepts/redaction-tokens]].

### 6. Tone & Style
- Checks for drift from a supplied `style_profile` (see [[concepts/style-profile-extraction]]).
- Flags bare-assertion clinical claims that should be hedged.
- Flags unexplained jargon for non-clinician audiences (see [[concepts/dual-audience-design]]).
- Prohibits emoji, first-person clinician voice, and self-praise.
- Related to [[concepts/clinical-communication-register]].

## Output Structure

Returns a structured verdict block:

```
REVIEW_VERDICT: <ready_to_sign_out | revise_before_signout | block_signout>

CRITICAL_ISSUES:   — PHI, fabricated scores, validity language errors (block sign-out)
MAJOR_ISSUES:      — Missing raters, missing validity statements (revise before signout)
MINOR_ISSUES:      — Tone drift, soft hedging failures
OK_AXES:           — Axes that passed cleanly
STATS:             — files_reviewed, csv_rows, domains_in_data, domains_in_narrative
```

## Hard Rules

- **Read-only**: Never calls Edit, Write, or any modifying tool.
- **Cite specifics**: Every finding must include a file path + line number or CSV row index.
- **Quote sparingly**: ≤15 words from patient content; PHI replaced with `[…]`.
- **No diagnostic second-guessing**: Scope is consistency, completeness, and language hygiene only.
- **Missing inputs**: If CSV or `qmd_dir` is unreadable, immediately returns `block_signout`.

## Related Concepts
- [[concepts/multi-agent-orchestration]]

- [[concepts/neuropsychological-assessment-pipeline]] — The broader orchestration context this agent operates within
- [[concepts/validity-language]] — Standards for hedging effort and validity findings
- [[concepts/phi-data-handling]] — Handling of protected health information
- [[concepts/pii-redaction-pipelines]] — Patterns and tokens for PHI detection and redaction
- [[concepts/redaction-tokens]] — Placeholder token conventions
- [[concepts/report-review-qa]] — Quality assurance processes for clinical report review
- [[concepts/modular-report-architecture]] — Structure of the domain-based narrative files
- [[concepts/subagent-architecture]] — How parallel agents like this reviewer integrate into multi-agent pipelines
- [[concepts/neuropsychological-reporting]] — Alignment between quantitative data and qualitative claims
- [[concepts/clinical-report-structure]] — Organization of neuropsychological report components