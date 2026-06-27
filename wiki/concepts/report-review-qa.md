---
sources: [summaries/agentic-workflows.md, summaries/Luria_AI_Q4_Investor_Memo_2026.md, summaries/agent-team.md, summaries/2026-04-26-cingulate-agent-team-design.md, summaries/customization.md, summaries/report-generator.md, summaries/report_body.md, summaries/clinical-validity-reviewer.md]
brief: Systematic, structured verification of clinical neuropsychological reports before human sign-off in automated pipelines.
---

# Report Review and Quality Assurance in Clinical Pipelines

Report review and quality assurance (QA) in clinical neuropsychological pipelines refers to the systematic, structured verification of draft reports before they are delivered to referral sources or filed. QA processes ensure that narrative claims are consistent with underlying data, that protected health information is handled correctly, that validity language meets professional standards, and that the final document is fit for its intended audience.

In automated or semi-automated pipelines, QA is often implemented as a dedicated read-only agent or subprocess — either running in parallel with report generation or as an explicit terminal stage. The [[summaries/agent-team]] runbook, for example, codifies QA as the fifth and final stage (`cingulate-quality-reviewer`) in a five-stage pipeline, producing a structured `issue_list.md` with severities (`blocker | major | minor | nit`) before any human sign-off.

## Why Dedicated QA Matters

Neuropsychological reports integrate quantitative test scores, qualitative clinical interpretation, premorbid context, and validity findings into a single document intended for multiple audiences (clinicians, educators, legal parties, patients). This complexity creates several failure modes:

- **Score–narrative mismatches**: A writer agent or human clinician may apply the wrong descriptive range label to a score.
- **Missing domains**: A subdomain present in the data may be omitted from the narrative without documentation.
- **Validity language errors**: Over-certain or under-hedged language around effort and performance validity can have legal and clinical consequences.
- **PHI leaks**: Template substitution failures or copy-paste errors can expose patient identifiers in final documents.
- **Tone drift**: Reports may shift register unexpectedly, especially when multiple agents or authors contribute sections.

Dedicated QA catches these issues systematically rather than relying on a single reviewer's attention.

## QA as a Pipeline Stage

In the Cingulate Agent Team (see [[summaries/agent-team]] and [[summaries/2026-04-26-cingulate-agent-team-design]]), QA is implemented as `cingulate-quality-reviewer`, a subagent wrapping the `luria-quality-review` skill. It receives the rendered PDF produced by the `cingulate-report-writer` stage, uses `pdftotext` to extract content, and emits an `qa/issue_list.md` to the per-patient workspace. Unlike the other stages, the QA subagent performs no R calls — it operates entirely through `pdftotext` and heuristics.

The orchestrator halts on a `BLOCKED` or `error` status if QA cannot run (e.g., `pdftotext` not installed), ensuring QA is never silently skipped. Because the orchestrator does not auto-retry, any QA failure surfaces immediately to the human operator for manual review and re-dispatch.

Critically, the team **never approves a report** — that is always a human decision. The QA stage's role is to surface issues at graded severity levels so that a human reviewer knows exactly what to address before sign-off. Blockers must be resolved; major issues require revision; minor issues and nits are advisory.

The [[concepts/agent-pipeline-state-management]] pattern used by the orchestrator (`state.json`) ensures QA status is persisted and resumable — if QA fails, the operator can fix the environment and re-dispatch without rerunning earlier stages. QA status follows the same `pending | in_progress | done | error` lifecycle as all other stages, and failures write a reason field plus stack trace to `logs/qa.log`.

## The Clinical Validity Reviewer Pattern

The [[summaries/clinical-validity-reviewer]] document describes a concrete implementation of this pattern. The agent:

1. **Reads** a score CSV (long-format, cingulate schema) and a directory of draft `.qmd` narrative files.
2. **Checks six axes**: completeness, validity language, premorbid context, score–narrative consistency, PHI leaks, and tone/style.
3. **Returns a structured punch list** with a three-tier verdict (`ready_to_sign_out`, `revise_before_signout`, `block_signout`) and categorized findings (CRITICAL, MAJOR, MINOR, OK).
4. **Never modifies files** — all output is advisory.

This read-only design means QA does not introduce new errors and cannot corrupt the document it is reviewing.

## Key QA Axes

### Completeness Checking
Every data domain must map to a narrative section. Multi-rater instruments (e.g., parent vs. teacher BRIEF ratings) must each have coverage. This is closely related to [[concepts/modular-report-architecture]], where domains are independently authored sections.

### Validity Language Compliance
The [[concepts/validity-language]] standards require that effort and performance validity findings be expressed with appropriate epistemic hedging. QA enforces the rule that terms like "malingering" or "feigned" are only used when positive validity findings are present, and that equivocal findings are never described with certainty.

### Score–Narrative Consistency
Every qualitative descriptor in the narrative (e.g., "Average", "Low Average") must match the `range` field in the corresponding CSV row. This is a cross-check between the [[concepts/neuropsychological-test-scores]] data layer and the [[concepts/narrative-report-generation]] output.

### PHI Detection
Regex-based scanning for date-of-birth patterns, SSNs, MRNs, phone numbers, and real names enforces the [[concepts/phi-data-handling]] and [[concepts/pii-redaction-pipelines]] standards. Patient content must use placeholder tokens (`[PATIENT]`, `[CLINICIAN]`, `[FACILITY]`) rather than literal identifiers. See also [[concepts/redaction-tokens]].

### Premorbid Context Verification
QA checks that the narrative situates current performance against estimated premorbid functioning — a standard requirement in [[concepts/neuropsychological-reporting]] that is easily omitted in templated or agent-generated drafts.

### Tone and Style Auditing
When a style profile is available (see [[concepts/style-profile-extraction]]), the reviewer samples the narrative voice and flags drift. It also enforces register rules relevant to [[concepts/clinical-communication-register]] and [[concepts/dual-audience-design]]: no jargon without explanation for lay readers, no first-person clinician voice, no bare-assertion diagnostic claims.

## Output Structure

A well-designed QA output is:

- **Tiered**: Critical/blocker issues block sign-out; major issues require revision; minor issues and nits are advisory.
- **Cited**: Every finding references a specific file path and line number (or CSV row index).
- **Bounded**: Quotes are capped at 15 words; PHI within quotes is replaced with `[…]`.
- **Scoped**: QA does not second-guess clinical diagnostic judgment — it enforces consistency, completeness, and language hygiene only.

In the Cingulate Agent Team design, QA output is written to `qa/issue_list.md` within the per-patient workspace, following the same absolute-path convention used by all other stage logs and state entries.

## Edit Protection and QA Interaction

The [[concepts/edit-protection-pattern]] used in the pipeline (adding `# manual-edit` as the first line of a `.qmd` file) interacts with QA: manually edited files are skipped by the interpretation stage but still reviewed by QA. This means a human-authored section receives the same completeness and PHI checks as a machine-generated one, preserving the integrity of the review gate.

## Relationship to Broader Pipeline Architecture

Report QA sits at the terminal stage of the [[concepts/neuropsychological-assessment-pipeline]] and the [[concepts/multi-agent-orchestration]] workflow. It complements the [[concepts/subagent-architecture]] pattern where specialized agents handle extraction, scoring, narrative writing, and review as distinct, sequential responsibilities.

In the Cingulate Agent Team, the QA subagent wraps the `luria-quality-review` skill from [[concepts/luria-skills]], following the same two-layer convention as all other cingulate subagents: a thin, generic skill describes *what* quality review does, while the cingulate-specific subagent binds it to this repo's file conventions and workspace layout. The [[concepts/per-patient-workspace]] structure — with `qa/issue_list.md` as a first-class output path — ensures QA findings are co-located with all other stage artifacts and are visible to both the orchestrator and the human reviewer.

QA agents also depend on upstream outputs: the score CSV produced by pdf score extraction and clinical PDF assessment pipelines, and the `.qmd` narrative files produced within the [[concepts/quarto]]-based documentation-as-code system.

## Summary

Report review and QA in clinical pipelines is not a bureaucratic afterthought — it is a structural safeguard that catches the failure modes inherent in complex, multi-source document assembly. Whether implemented as the read-only, axis-structured agent described in [[summaries/clinical-validity-reviewer]] or as the explicit final stage of the Cingulate Agent Team in [[summaries/2026-04-26-cingulate-agent-team-design]], the principle is the same: QA provides a reproducible, auditable quality gate that scales with automated report generation, while preserving human authority over final sign-off.

See also: [[summaries/report_body]]

See also: [[summaries/report-generator]]

See also: [[summaries/customization]]

See also: [[summaries/2026-04-26-cingulate-agent-team-design]]

See also: [[summaries/Luria_AI_Q4_Investor_Memo_2026]]

See also: [[summaries/agentic-workflows]]