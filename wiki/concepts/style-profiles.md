---
sources: [summaries/Apply-to-Y-Combinator-JWT.md, summaries/neurobehav.prompt.md, summaries/DEPENDENCIES.md, summaries/full-pipeline.md, summaries/customization.md]
brief: JSON configuration objects encoding a clinician's writing voice, tone, rules, and structure for AI report generation.
---

# Clinical Style Profiles for Report Generation

A **style profile** is a JSON configuration object that encodes the distinctive writing conventions of a clinician or population context, enabling an AI agent to generate neuropsychological reports that match a specific voice, tone, and structural pattern. Style profiles are the primary customization mechanism of the [[concepts/narrative-report-generation]] workflow and are central to the [[summaries/soul-style-agent]] architecture.

## What a Style Profile Contains

A style profile captures several dimensions of clinical writing:

### Voice and Tone
- `voice` — A natural-language description of the authorial persona (e.g., "Professional clinical analyst with developmental focus").
- `tone` — The affective register (e.g., "Objective yet empathetic, strength-based").

### Structural Conventions
- `structure_patterns` — An ordered list of compositional rules (e.g., "Begin with developmental history context", "Present cognitive findings before behavioral").

### Explicit Rules
- `do_rules` — Enforced inclusions, written in imperative form (e.g., "USE PERSON-FIRST LANGUAGE ALWAYS", "Include percentile ranks with all standard scores").
- `avoid_rules` — Enforced exclusions (e.g., "NEVER use definitive diagnostic statements", "NEVER include casual contractions").

See [[summaries/customization]] for full JSON examples.

## How Profiles Are Created

Profiles are generated through a two-step pipeline:

1. **Index building** — A corpus of existing reports is parsed and stored in a [[concepts/sqlite-as-vector-store]] database using [[concepts/rag-chunking]] to segment text into retrievable units.
2. **Style training** — The agent analyzes the indexed corpus to extract recurring linguistic patterns, producing a `.json` profile file.

This process is described in detail in [[summaries/style-trainer]] and [[summaries/style-training-to-report-drafting]].

## Profile Scopes

### Per-Clinician Profiles
Each clinician can maintain a dedicated profile trained solely on their own historical reports, preserving individual idiosyncrasies in phrasing, section ordering, and interpretive language. This is the finest-grained customization level.

### Per-Population Profiles
Broader profiles can be trained for entire practice populations:
- **Pediatric** — Emphasizes developmental history, age-normed language, and parent-facing recommendations.
- **Adult** — Standard neuropsychological framing with adult normative comparisons.
- **Forensic** — Heightened objectivity, avoidance of diagnostic overclaiming, and precise evidentiary language (see [[concepts/forensic-neuropsychological-evaluation]]).

### Manual Editing
Profiles can be hand-edited after generation for fine-grained control — for example, adding domain-specific `do_rules` or restructuring `structure_patterns` without retraining from scratch.

## Profile Usage at Generation Time

During [[concepts/narrative-report-generation]], the profile is passed to the report-writing agent alongside retrieved context chunks. The agent uses the profile to:
- Constrain vocabulary and tone.
- Enforce structural ordering of sections.
- Apply do/avoid rules as generation guardrails.

Retrieval parameters (`--top-k`, `--temperature`) interact with the profile: lower temperature preserves closer adherence to profile conventions, while higher top-k provides richer stylistic exemplars. See [[summaries/customization]] for parameter combinations.

## Quality Assurance

Profiles can be evaluated through A/B testing — generating parallel drafts from two profiles against the same case input and comparing outputs with `diff`. Automated post-generation checks can validate that profile `avoid_rules` (e.g., no contractions, no unfilled data markers) were respected. See [[concepts/report-review-qa]].

## Related Concepts
- [[concepts/style-profile-extraction]] — The process of deriving profile fields from a report corpus.
- [[concepts/retrieval-augmented-generation]] — The broader RAG framework within which profiles operate.
- [[concepts/rag-chunking]] — Chunking strategy affects what stylistic patterns are captured in the index.
- [[concepts/clinical-report-structure]] — The structural conventions that profiles encode.
- [[concepts/clinical-communication-register]] — The clinical voice and tone dimensions that profiles formalize.
- [[concepts/neuropsychological-reporting]] — The broader reporting context.
- [[summaries/0004-soul-style-profile-json]] — ADR describing the profile JSON format.
- [[summaries/soul-style-agent]] — The agent that consumes style profiles.


See also: [[summaries/full-pipeline]]

See also: [[summaries/DEPENDENCIES]]

See also: [[summaries/neurobehav.prompt]]

See also: [[summaries/Apply-to-Y-Combinator-JWT]]