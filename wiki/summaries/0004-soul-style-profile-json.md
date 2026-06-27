---
doc_type: short
full_text: sources/0004-soul-style-profile-json.md
---

# ADR 0004: JSON Style Profiles

**Date:** 2025-01-20
**Status:** Accepted

## Overview

This Architecture Decision Record (ADR) defines the use of **JSON files** as the storage format for [[concepts/style-profile-extraction]], enabling writing style characteristics to persist across sessions in the SOUL system. See also [[concepts/architecture-decision-records]] for context on how ADRs are used in this project.

## Problem

The system needed a way to persist learned writing styles so that style analysis does not need to be repeated every session. The stored data must capture voice, tone, structure, and phrasing preferences.

## Schema Definition

The style profile JSON schema contains six top-level fields:

| Field | Purpose |
|---|---|
| `voice` | Overall voice descriptor (e.g., "Professional clinical") |
| `tone` | Tone characterization (e.g., "Objective yet empathetic") |
| `structure_patterns` | List of structural conventions used in documents |
| `typical_phrases` | Approved example phrasings |
| `do_rules` | Explicit style rules to follow |
| `avoid_rules` | Explicit style rules to avoid |

This schema is intentionally flat and extensible — new fields can be added without breaking existing consumers.

## Generation Process

The profile is produced via a [[concepts/retrieval-augmented-generation]] pipeline:

1. Retrieve style-exemplar chunks via RAG query
2. Prompt LLM to analyze writing patterns across chunks
3. LLM returns a compact JSON profile
4. Save to `report_style_profile.json`

This ties profile quality directly to LLM analysis quality — a noted limitation.

## Usage in Prompts

The JSON profile is embedded directly into generation prompts using Python's `json.dumps()`, enforcing style at inference time:

```python
report_prompt = f"""
STYLE PROFILE (must follow):
{json.dumps(profile, indent=2)}

USER TASK:
{args.prompt}
...
"""
```

This approach makes the style profile a first-class prompt engineering artifact, directly shaping [[concepts/narrative-report-generation]].

## Trade-offs

### Positive
- **Human Readable**: Clinicians can review and manually edit profiles
- **Version Control Friendly**: JSON diffs cleanly in git
- **Language Agnostic**: Any tool or language can consume the file
- **Extensible**: Open schema allows new fields

### Negative
- **No Validation**: Schema is informal; no enforced contract
- **LLM Dependency**: Profile quality is only as good as the LLM's analysis
- **Static**: Profiles do not self-update; require re-running `train-style`

## Key References

- `neuro_report_style_agent.py:238-279` — `train-style` command implementation
- `neuro_report_style_agent.py:282-319` — `write-report` command implementation

## Related Concepts
- [[concepts/clinical-data-privacy]]
- [[concepts/persistent-memory]]

- [[concepts/style-profile-extraction]] — Cross-document synthesis of style persistence patterns
- [[concepts/retrieval-augmented-generation]] — Retrieval-Augmented Generation used in profile generation
- [[concepts/narrative-report-generation]] — How profiles shape generated clinical reports
- [[concepts/neuropsychological-reporting]] — The clinical reporting context this system serves
- [[concepts/yaml-configuration]] — Alternative configuration format considered for structured data storage