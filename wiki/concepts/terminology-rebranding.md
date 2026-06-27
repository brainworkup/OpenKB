---
sources: [summaries/2026-06-26-2133-plan.md]
brief: The process of systematically updating feature names across codebases and documentation to maintain consistent branding.
---

# Terminology Rebranding in Clinical Software

Terminology rebranding refers to the deliberate, systematic process of renaming a software feature or concept and propagating that new name consistently across all codebases, documentation, prompts, and user-facing materials. In clinical software, this process is especially important because inconsistent naming can create confusion for clinicians, developers, and automated pipelines alike.

## Why Terminology Rebranding Matters

In clinical and neuropsychological software, feature names carry meaning beyond simple labels — they communicate the nature of an assessment, its methodology, and its regulatory or clinical context. When a feature is renamed:

- **End users** (clinicians, report readers) must encounter the new name consistently to build accurate mental models.
- **Developers** must ensure that code, configuration files, prompts, and templates all use the canonical term.
- **Automated pipelines** (e.g., parsers, extractors, RAG systems) may match on string patterns; stale terminology can cause silent failures.
- **Documentation** must reflect the current state of the product, not a historical artifact.

## The Documentation Gap Problem

A common failure mode in rebranding is updating the codebase first and documentation second — or never. This creates a *documentation gap*: the software behaves according to the new name, but the docs still describe the old one. This gap causes:

- Onboarding confusion for new developers or contributors.
- Incorrect search results when querying documentation.
- Inconsistency between user manuals, changelogs, and in-app labels.

The plan described in [[summaries/2026-06-26-2133-plan]] is a direct response to this problem: the codebase had already been updated from "Status Exam" to "Neurobehavioral Exam (STT)", but the `docs/` and `notes/` directories still contained legacy references.

## Key Principles

### 1. Canonical Name Definition
Establish a single authoritative form of the new name before making changes. In the example from [[summaries/2026-06-26-2133-plan]], the canonical name is **Neurobehavioral Exam (STT)**, which must replace all prior variants:
- "Status Exam"
- "Neurobehavioral Status Exam"
- "Neurobehavioral Exam" (without the STT qualifier)

### 2. Exhaustive Search
Use automated scanning (e.g., `grep`, IDE search, or script-based discovery) to find every occurrence of the old term across all file types — `.md`, `.qmd`, `.yaml`, `.json`, prompts, templates, and scripts.

### 3. Context-Sensitive Replacement
Not every occurrence is a simple find-and-replace. Context matters:
- A phrase like "Neurobehavioral Exam (STT) transcript" preserves the content-type distinction.
- A section header may need rephrasing for grammatical correctness.
- Variable names in code may follow a different convention (e.g., `snake_case`).

### 4. Staged Verification
After replacement, verify that:
- No old terminology remains in target files.
- The new terminology reads naturally in context.
- Downstream systems (parsers, RAG pipelines, prompt templates) still function correctly.

## Clinical Software Considerations

In neuropsychological and clinical AI systems, terminology rebranding intersects with several technical domains:

- **[[concepts/clinical-nlp-pipelines]]**: String matching on feature names may be embedded in parsing logic.
- **[[concepts/neuropsychological-prompt-configuration]]**: Prompt templates may reference the old feature name explicitly.
- **[[concepts/narrative-report-generation]]**: Report section headers and boilerplate text must use the canonical name.
- **[[concepts/neurobehavioral-status-exam]]**: The specific clinical assessment being rebranded in this case.
- **[[concepts/documentation-as-code]]**: Treating docs as versioned artifacts makes rebranding changes trackable via version control.
- **[[concepts/luria-neuropsych-pipeline]]**: The broader pipeline in which this feature operates.

## Process Template

| Step | Action |
|------|--------|
| 1 | Define the canonical new name |
| 2 | Scan all directories for old terms |
| 3 | Categorize occurrences by context |
| 4 | Perform context-sensitive replacements |
| 5 | Verify no legacy terms remain |
| 6 | Update changelogs and decision records |
| 7 | Communicate changes to stakeholders |

## Related Concepts

- [[concepts/documentation-as-code]] — Versioned, structured docs make rebranding changes auditable.
- [[concepts/neurobehavioral-status-exam]] — The specific assessment subject to rebranding here.
- [[concepts/clinical-nlp-pipelines]] — Pipelines that may embed feature name strings.
- [[concepts/neuropsychological-prompt-configuration]] — Prompt files that must reflect the new name.
- [[concepts/architecture-decision-records]] — May capture the rationale for the rename.
- [[concepts/luria-neuropsych-pipeline]] — The system context in which this rebranding occurs.