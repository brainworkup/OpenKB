---
sources: [summaries/ARCHITECTURE.md, summaries/2026-06-26-2133-plan.md]
brief: Systematic updating of feature names across codebases and documentation to maintain consistent branding
---

<mark>Terminology Rebranding</mark> in Clinical Software

Why Terminology Rebranding Matters

In clinical and neuropsychological software, feature names carry meaning beyond simple labels -- they communicate the nature of an assessment, its methodology, and its regulatory or clinical context. When a feature is renamed: 

End users (clinicians, report readers) must encounter the new name consistently to build accurate mental models.
Developers must ensure that code, configuration files, prompts, and templates all use the canonical term.
Automated pipelines (e.g., parsers, extractors, RAG systems) may match on string patterns; stale terminology can cause silent failures.
Documentation must reflect the current state of the product, not a historical artifact.

The Documentation Gap Problem

A common failure mode in rebranding is updating the codebase first and documentation second -- or never. This creates a *documentation gap*: the software behaves according to the new name, but the docs still describe the old one. This gap causes: 
Onboarding confusion for new developers or contributors.
Incorrect search results when querying documentation.
Inconsistency between user manuals, changelogs, and in-app labels.

Key Principles

1. Canonical Name Definition Establish a single authoritative form of the new name before making changes.
2. Exhaustive Search Use automated scanning (e.g., `grep`, IDE search, or script-based discovery) to find every occurrence of the old term across all file types -- `.md`, `.qmd`, `.yaml`, `.json`, prompts, templates, and scripts.
3. Context-Sensitive Replacement Not every occurrence is a simple find-and-replace. Context matters: 
A phrase like "Neurobehavioral Exam (STT) transcript" preserves the content-type distinction.
A section header may need rephrasing for grammatical correctness.
Variable names in code may follow a different convention (e.g., `snake_case`).

4. Staged Verification After replacement, verify that: 
No old terminology remains in target files.
The new terminology reads naturally in context.
Downstream systems (parsers, RAG pipelines, prompt templates) still function correctly.

Clinical Software Considerations

In neuropsychological and clinical AI systems, terminology rebranding intersects with several technical domains: 
1. [[concepts/clinical-nlp-pipelines]] -- String matching on feature names may be embedded in parsing logic.
2. [[concepts/neuropsychological-prompt-configuration]] -- Prompt templates may reference the old feature name explicitly.
3. [[concepts/narrative-report-generation]] -- Report section headers and boilerplate text must use the canonical name.
4. [[concepts/neurobehavioral-status-exam]] -- The specific clinical assessment being rebranded in this case.
5. [[concepts/documentation-as-code]] -- Treating docs as versioned artifacts makes rebranding changes trackable via version control.
6. [[concepts/luria-neuropsych-pipeline]] -- The broader pipeline in which this feature operates.

Process Template

| Step | Action | 
| 1 | Define the canonical new name | 
| 2 | Scan all directories for old terms | 
| 3 | Categorize occurrences by context | 
| 4 | Perform context-sensitive replacements | 
| 5 | Verify no legacy terms remain | 
| 6 | Update changelogs and decision records | 
| 7 | Communicate changes to stakeholders | 

Related Concepts

1. [[concepts/documentation-as-code]] -- Versioned, structured docs make rebranding changes auditable.
2. [[concepts/neurobehavioral-status-exam]] -- The specific assessment subject to rebranding here.
3. [[concepts/clinical-nlp-pipelines]] -- Pipelines that may embed feature name strings.
4. [[concepts/neuropsychological-prompt-configuration]] -- Prompt files that must reflect the new name.
5. [[concepts/architecture-decision-records]] -- May capture the rationale for the rename.
6. [[concepts/luria-neuropsych-pipeline]] -- The system context in which this rebranding occurs.

New information from document "ARCHITECTURE" (summarized above) should be integrated into this page. Rewrite the full page incorporating the new information naturally -- do not just append. Preserve the existing structure and intent of the page. For wikilinks in the rewrite, follow the whitelist rules from the message above: keep links whose target is in the whitelist, convert any existing links whose target is NOT in the whitelist to plain text, and do not invent new wikilink targets. Return a JSON object with two keys:

## Related Documents
- [[summaries/ARCHITECTURE]]
