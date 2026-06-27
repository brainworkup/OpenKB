---
sources: [summaries/PERMANENT_SOLUTION_SUMMARY.md, summaries/AGE_OVERRIDE_GUIDE.md, summaries/copilot-instructions.md, summaries/CLAUDE.md, summaries/agent-team.md, summaries/2026-04-26-cingulate-agent-team-design.md, summaries/neuropsych-narrative-writer.md]
brief: Safeguard preventing automated pipeline stages from silently overwriting human-edited clinical files.
---

# Edit-Protection Pattern for AI-Generated Clinical Files

The **edit-protection pattern** is a safeguard mechanism used in AI-assisted document generation workflows where human experts may hand-edit files produced by an automated stage. Its core principle: **never silently overwrite a file that contains content not derivable from the current data source**.

This pattern is particularly important in clinical and neuropsychological reporting, where a clinician may refine AI-generated prose — adding nuance, correcting tone, or inserting case-specific observations — after the initial automated draft is produced.

## The Problem

In multi-stage pipelines like the Luria neuropsychological pipeline and the `cingulate` R package pipeline, an AI agent or automated stage may be re-run on updated data (e.g., after additional test scores are extracted). Without protection, re-running the narrative writer would **destroy clinician edits** that were made to the previously generated `.qmd` files — edits that are not recoverable from the CSV data alone.

The `cingulate` package explicitly acknowledges this risk: its two-stage rendering process (text QMD generated first, then a shell QMD that includes it and adds tables/plots) means regenerating QMDs can overwrite manual edits. The guidance is to check whether a target file has been hand-edited before regenerating.

The same risk surfaces in [[concepts/multi-agent-orchestration]] contexts: when an orchestrator re-dispatches a stage subagent after a data update or error recovery, it has no inherent awareness that a human may have touched the output files in the interim. This is directly addressed in the Cingulate Agent Team design ([[summaries/2026-04-26-cingulate-agent-team-design]]) and its operator runbook ([[summaries/agent-team]]), both of which include the edit-protection convention in operator-facing documentation rather than only in internal subagent prompts.

## How It Works

### General Form

The pattern as defined in [[summaries/neuropsych-narrative-writer]] follows three steps:

1. **Read before write**: Before generating or overwriting any existing `_text.qmd` file, the agent reads the current file contents.
2. **Detect foreign content**: The agent checks whether the existing file contains content that could not have been produced from the current data source (i.e., the extractor's CSV). If such content is present, it is assumed to be a clinician hand-edit.
3. **Safe fallback behavior**: Instead of overwriting, the agent:
   - Appends the new AI-generated draft as a **comment block** (`<!-- DRAFT: ... -->`) below the existing content.
   - Reports an `EDIT_PROTECTION_HIT` in its final output summary, naming the affected file.

This approach preserves human work while still delivering the updated draft in a discoverable, non-destructive location.

### Marker-Based Form (Cingulate Agent Team)

The Cingulate Agent Team introduces a lightweight, explicit variant: a **`# manual-edit` marker on line 1** of any `_02-XX_*_text.qmd` file. Each stage subagent — specifically `cingulate-interpretation` — is required to check for this marker before regenerating the file. If present, the subagent **skips the file entirely and logs a warning** rather than appending or overwriting.

This convention is described both in the full design document ([[summaries/2026-04-26-cingulate-agent-team-design]]) and in the operator runbook ([[summaries/agent-team]]), making it visible to whoever is managing a live run — not just to the subagents themselves. The runbook's inclusion of this convention signals that edit-protection is considered an **operational concern**, not merely an internal implementation detail.

The marker-based approach is simpler and more deterministic than heuristic content-diffing. It places the edit-protection signal squarely in the hands of the clinician or reviewer: adding `# manual-edit` to line 1 is the explicit declaration that this file is now human-owned. The convention is restated in every stage subagent's definition because each runs in its own context without access to shared state beyond the [[concepts/per-patient-workspace]].

This design choice is consistent with the broader Cingulate scaffolding philosophy: because subagents receive only a self-contained context slice (workspace path, patient slug, age group, and the relevant state section) rather than the full conversation history, all operational conventions — including edit-protection — must be explicitly restated in each subagent's definition rather than inherited from a shared context. This is noted in the Cingulate design document as a deliberate architectural decision tied to the [[concepts/subagent-architecture]] pattern.

### Cingulate R Package Form

The `cingulate` R package implements edit-protection as an integral part of the domain QMD generation workflow. The package's domain numbering convention (`01_iq`, `02_academics`, `03_verbal`, etc.) is explicitly stable — clinicians are warned not to renumber domains casually — in part because a stable naming scheme makes it feasible to check specific named files for hand-edits before regenerating. The `generate_domain_text_qmd()` function (which writes `_02-XX_domain_text.qmd` files) is the primary write target that requires protection.

Multi-rater domain files (e.g., for ADHD and emotion domains, which produce separate per-rater files for self/parent/teacher) are each checked independently — protection is not applied at the domain level but at the individual file level. The check for rater-specific data existence (`check_rater_data_exists()` / `check_domain_raters()`) is a prerequisite step that runs before any file write, and it naturally gates the regeneration path.

## Output Signaling

All variants require the agent or pipeline to surface edit-protection events explicitly. In the narrative writer pattern:

```
EDIT_PROTECTION_HITS: _02-06_executive_text.qmd (existing hand-edits detected; draft appended as comment)
```

In the Cingulate scaffolding, the warning is written to `logs/<stage>.log` and surfaced by the orchestrator in its final summary. The operator runbook instructs practitioners to check stage logs when a stage takes longer than expected or returns `DONE_WITH_CONCERNS` — edit-protection warnings would appear in this same log stream. This gives the clinician or workflow orchestrator a clear audit trail of where human review is required before the next `quarto render`.

The Cingulate orchestrator does **not** auto-retry on blocked or errored stages. Failures — including those triggered by edit-protection events — halt the chain and surface to the operator for manual review and re-dispatch. This conservative posture is part of the approval-gate model: the operator reviews before any subagent is invoked against a real case, and the same manual-review ethos extends to mid-run interventions.

## Why It Matters in Clinical Contexts

In [[concepts/neuropsychological-reporting]], prose carries significant clinical and legal weight. A report narrative that has been reviewed and edited by a licensed neuropsychologist represents a professional judgment — not merely a data transformation. Automated pipelines must respect this boundary.

The edit-protection pattern enforces a clear **human-in-the-loop** checkpoint: AI generates the first draft; the clinician owns the final version. The pipeline cannot inadvertently erase that ownership.

This also connects to [[concepts/phi-data-handling]]: if a clinician has added patient-specific observations that were not present in the upstream scrubbed data, overwriting those edits could create both a data integrity problem and a compliance risk. The Cingulate runbook reinforces this principle by noting that the team **never approves a report** — human sign-off is always required, and edit-protection is one mechanism that keeps the human meaningfully in control.

## Relationship to Broader Patterns

- **[[concepts/modular-report-architecture]]**: The per-domain `.qmd` file structure makes edit-protection tractable — each file is small and domain-scoped, so marker-checking or diff-detection is feasible. The `cingulate` package's domain QMD pattern is a direct instantiation of this.
- **[[concepts/narrative-report-generation]]**: The pattern applies specifically to AI-generated narrative files, not to data tables or plots rendered by the R layer.
- **[[concepts/clinical-report-structure]]**: The stable domain prefix numbering (`_02-01_iq`, `_02-06_executive`, etc.) means the agent always knows exactly which file to check before writing.
- **[[concepts/report-review-qa]]**: Edit-protection hits serve as QA signals, flagging files that need human reconciliation before final render.
- **[[concepts/fallback-strategy]]**: The comment-block append is a graceful degradation — the pipeline continues producing output without data loss.
- **[[concepts/per-patient-workspace]]**: The workspace-as-contract model means edit-protection checks are always scoped to a single patient directory, reducing the risk of cross-patient interference.
- **[[concepts/subagent-architecture]]**: Because each stage subagent runs in isolation without full conversation history, edit-protection conventions must be restated in every subagent definition rather than inherited from a shared context.
- **[[concepts/agent-pipeline-state-management]]**: The `state.json` file used by the Cingulate orchestrator tracks stage completion, enabling the resume-after-fix workflow — but edit-protection checks remain the responsibility of individual subagents, not the state machine.
- **[[concepts/luria-skills]]**: The luria-interpretation skill, wrapped by `cingulate-interpretation`, is the primary consumer of this pattern — it generates the per-domain QMD narratives that clinicians are most likely to hand-edit.
- **[[concepts/quarto]]**: The two-stage rendering model (text QMD → shell QMD → Quarto/Typst → PDF) creates the specific file targets that edit-protection must guard.
- **[[concepts/r6-class-architecture]]**: The `cingulate` package's R6-first design means domain generation logic is encapsulated in class methods, making it straightforward to add pre-write edit-protection checks at the class level.

## Implementation Variants

| Variant | Detection method | On hit | Best for |
|---|---|---|---|
| Heuristic content-diff | Check if content is derivable from current source data | Append draft as comment block | Pipelines without marker infrastructure |
| Marker-based (`# manual-edit`) | Check line 1 of file | Skip entirely, log warning | Explicit human control; simpler orchestration |
| Checksum/sidecar | Hash of last-generated output stored in `.lock` file | Configurable | High-reliability production pipelines |

The Cingulate Agent Team currently uses the **marker-based** approach as the simplest option compatible with its scaffolding-only phase. The heuristic approach is used by the narrative writer agent. A checksum/sidecar implementation is noted as a more robust future option for production runs.

## Implementation Notes

- The heuristic detection method ("content not derivable from current data") is inherently imperfect. A robust implementation may use checksums, generation metadata embedded as comments, or a sidecar `.lock` file tracking the last-generated hash.
- In the `cingulate` R package, the canonical input location (`data-raw/csv/`) is gitignored, meaning the source data cannot be reconstructed from version control alone. This makes edit-protection even more important: there is no VCS fallback for reconstructing what the AI originally generated from a specific dataset.
- In the Cingulate pipeline, the orchestrator surfaces warnings and failures to the user rather than silently continuing. The operator runbook explicitly instructs practitioners to re-dispatch only after manually resolving the root cause — no auto-retry on `BLOCKED` status.
- The pattern applies equally to multi-rater domain files (e.g., `_02-09_adhd_parent_text.qmd`) — each rater file is checked independently.
- The `# manual-edit` marker convention must be documented in each stage subagent's own prompt, not in a shared config, because subagents receive only a context slice and not the full project configuration.
- The Cingulate design document's inclusion of this convention in operator-facing documentation (not just internal subagent definitions) is notable: it makes edit-protection a first-class operational concern that human practitioners are expected to understand and manage.
- A synthetic fixture case under `output/_fixture/` is recommended as a next step for smoke-testing the full chain — including edit-protection behavior — without touching real PHI. This would allow validation that marker detection and skip-with-warning logic works correctly before any live run.
- Two competing main entry points exist in the `cingulate` package (`Cingulate2MainR6.R` and `cingulateMainR6`). Any edit-protection implementation must account for which entry point is active, as they may invoke domain generation methods differently.

## Related Documents
- [[summaries/2026-04-26-cingulate-agent-team-design]]
- [[summaries/agent-team]]
- [[summaries/neuropsych-narrative-writer]]
- [[summaries/CLAUDE]]

See also: [[summaries/copilot-instructions]]

See also: [[summaries/AGE_OVERRIDE_GUIDE]]

See also: [[summaries/PERMANENT_SOLUTION_SUMMARY]]