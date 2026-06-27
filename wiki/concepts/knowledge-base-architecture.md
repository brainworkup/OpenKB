---
sources: [summaries/Luria_AI_Q4_Investor_Memo_2026.md, summaries/redesign_20260623110910.md, summaries/redesign_20260623110817.md, summaries/SESSION_SUMMARY.md, summaries/DIAGNOSIS_FIX_SUMMARY.md, summaries/AUTISM_RAG_SYSTEM_DOCUMENTATION.md, summaries/2026-02-11-this-session-is-being-continued-from-a-previous-co.md, summaries/README.md]
brief: Tiered architecture separating curated reference wikis from shared auto-ingested clinical resources for clinical AI.
---

# Knowledge Base Architecture for Clinical AI Systems

A **knowledge base architecture for clinical AI** separates reference material into structured tiers, each optimized for its retrieval pattern, query frequency, and data sensitivity. This design is exemplified by the [[summaries/README_luria]] for Luria KB, which supplies diagnostic criteria, scoring tables, validity language, and report-writing standards to neuropsychology agents before they reason or write.

## Core Problem

Clinical AI systems face competing pressures:
- **Breadth:** Agents need diverse reference knowledge (diagnostic criteria, scoring norms, compliance rules, template formats)
- **Speed and cost:** Not all knowledge is queried equally; some content is needed on every call, some rarely
- **Privacy:** Patient-identifiable content must be handled differently from public reference material
- **Quality:** Different content types suit different retrieval methods (chunked embeddings vs. long-context)

A monolithic knowledge base — one corpus, one retrieval method — fails at least one of these dimensions. A tiered architecture addresses each explicitly.

## The Two-Tier Pattern

### Tier 1 — Curated Reference Wiki

A small (~40 notes, ~250 KB), hand-authored markdown knowledge base covering the concepts every report touches:
- Diagnostic criteria (DSM-5-TR, ICD-10)
- Score classification systems (Wechsler, Heaton norms)
- [[concepts/validity-language]] for performance validity and symptom validity testing
- Recommendations writing standards
- HIPAA/APA compliance requirements
- Base rates, Reliable Change Index (RCI)

**Retrieval method:** Long-context retrieval with prompt caching. The entire corpus fits in a single cached system prompt (~250 KB). Queries land on a warm cache; citations return as exact quoted spans from source notes. This avoids chunking artifacts and gives holistic cross-note answers.

**Query frequency:** Every report — this tier supplies ~10–15% of each draft that requires evidence grounding (diagnoses, recommendations, validity language).

**Organization:** 9 topic pillars + 31 concept notes, cross-linked with wikilinks in Obsidian-style markdown. See [[concepts/luria-neuropsych-pipeline]] for how these notes feed agent workflows.

### Tier 2 — Specialized Sub-Corpora

Purpose-specific stores queried sporadically by the agent that needs them:

| Sub-corpus | Purpose | Retrieval |
|------------|---------|----------|
| `store/typst/` | Vendored Typst docs for report template generation | Embeddings + vector search |
| `store/quarto/` | Vendored Quarto docs for R/Quarto pipelines | Embeddings + vector search |
| `store/shared_references/` | Shared clinical reference materials: behavioral observation guides, rating scales, assessments, research | Embeddings + vector search (legacy RAG pipeline) |

Each sub-corpus is a vendored sibling repository with its own `.git`. See [[concepts/retrieval-augmented-generation]] for the underlying retrieval patterns, and [[concepts/vector-search]] for the embedding-based approach used in Tier 2.

**Retrieval method:** [[concepts/hybrid-search-retrieval]] or embeddings + vector search. The embeddings tax is paid only where corpus size and heterogeneity demand it.

## Shared Reference Materials

The shared reference layer within Tier 2 is organized into several directories, all following the same core pattern: files placed here — PDF, DOCX, or plain text — are **automatically ingested whenever any patient's RAG system is rebuilt**, using descriptive filenames for reliable retrieval. This centralized design ensures every patient-facing system draws from the same evidence base without duplicating content.

### recommendations/

Stores evidence-based intervention recommendations and resources shared across **all** patient RAG systems. Any PDF, DOCX, or text file placed in this directory is automatically ingested when any patient's RAG system is rebuilt. Descriptive filenames are recommended. This directory was last updated 2026-01-30. It functions as a dedicated shared layer for intervention guidance, complementing `clinical_resources/guidelines`, `clinical_resources/dsm5`, and `clinical_resources/books` by focusing specifically on actionable, evidence-based recommendations rather than raw diagnostic or reference content. See [[summaries/README]] for the source README.

### clinical_resources/guidelines

Stores professional practice guidelines shared across all patient RAG systems. Any PDF, DOCX, or text file placed in this directory is automatically available to all patient-facing retrieval systems on the next rebuild. Descriptive filenames are recommended. This directory was last updated 2026-01-16. It functions as a peer to `clinical_resources/dsm5` and `clinical_resources/books`, contributing professional and regulatory practice standards that complement both diagnostic criteria and broader textbook references.

### clinical_resources/dsm5

Stores DSM-5 and related diagnostic reference materials shared across all patient RAG systems. Any PDF, DOCX, or text file placed in this directory is automatically available to all patient-facing retrieval systems on the next rebuild. Descriptive filenames are recommended. This layer was last updated 2026-01-16. This directory is a peer to `clinical_resources/books` and complements it with diagnosis-specific reference content aligned to DSM-5-TR criteria — the same criteria summarized in the Tier 1 wiki.

### clinical_resources/books

Stores clinical reference books and textbooks shared across all patient RAG systems. This directory is a key entry point for clinicians adding new reference material: any PDF, DOCX, or text file placed here is automatically available to all patient-facing retrieval systems on the next rebuild. Descriptive filenames are recommended. This layer was last updated 2026-01-30.

### assessments/behavioral_observations
Stores behavioral observation guides and [[concepts/behavioral-rating-scales]] shared across all patient RAG systems.

### assessments/manuals
Stores reference manuals for standardized assessments — such as the WISC (Wechsler Intelligence Scale for Children) and [[concepts/wais-iv]] — shared across all patient RAG systems.

### assessments/neurocognitive
Stores neurocognitive assessment materials and interpretation guides shared across all patient RAG systems.

All shared reference directories follow the same pattern: **centralized, automatically-ingested shared references consumed by per-patient systems**. This provides a stable common foundation beneath each individualized [[concepts/per-patient-workspace]] without duplicating content across patients.

## The Privacy Boundary

The two-tier split maps directly onto a privacy boundary — a defining feature of clinical KB architecture:

| Data Class | Tier | Cloud/Local | Rationale |
|------------|------|-------------|----------|
| Reference material (wiki, Typst/Quarto docs, diagnostic criteria, clinical books, DSM-5 resources, guidelines, behavioral observation guides, assessment manuals, neurocognitive materials, intervention recommendations) | Tier 1 & 2 public stores | Cloud APIs allowed | Public knowledge; no PHI |
| Patient data (case histories, score data, draft reports, PAI/MMPI profiles) | Outside KB entirely | **Local only** | HIPAA; PHI never leaves device |

This is the [[concepts/local-first-architecture]] principle applied at the **data layer**, not the feature layer. The KB holds *only* reference material. PHI-bearing content — exemplar reports, clinician voice profiles — lives in separate systems running against local models (Ollama/MLX). See [[concepts/phi-data-handling]] and [[concepts/clinical-data-privacy]] for related patterns.

## Agent Routing

Different agents query different tiers:

```
Diagnosis agent          → wiki (Tier 1) + dsm5/ (Tier 2)
Recommendations agent    → wiki (Tier 1) + recommendations/ (Tier 2)
Report-writing agent     → typst/ or quarto/ (Tier 2)
Behavioral agent         → shared_references/ (Tier 2)
Neurocognitive agent     → assessments/neurocognitive/ (Tier 2)
Clinical books agent     → clinical_resources/books/ (Tier 2)
DSM-5 agent              → clinical_resources/dsm5/ (Tier 2)
Guidelines agent         → clinical_resources/guidelines/ (Tier 2)
```

This agent-to-corpus routing is a form of [[concepts/role-based-llm-routing]] applied to retrieval rather than generation. See [[concepts/multi-agent-orchestration]] and [[concepts/luria-overview]] for the broader agent pipeline context.

## Rationale: Why Not One Tier?

- **Long-context + caching** is faster and cheaper for small, frequently-queried corpora — no embedding indexing, no chunking, no vector similarity computation at query time
- **Embeddings + vector search** scales to large, heterogeneous corpora that cannot fit in context — necessary for full Typst/Quarto documentation sets, clinical reference books, DSM-5 materials, professional practice guidelines, intervention recommendations, and shared clinical references
- Mixing the two in one index would either over-pay (embedding a 250 KB wiki) or under-perform (trying to long-context a multi-MB doc store)

The right-sizing principle: match retrieval method to corpus size, query frequency, and required answer type.

## Implementation Notes

- CLI entry point: `kb-ask` (via `kb_ask.py`) for Tier 1 queries; returns cited answers with wikilinks pointing back to `wiki/notes/`
- Shared reference directories (e.g., `recommendations/`, `clinical_resources/guidelines`, `clinical_resources/books`, `clinical_resources/dsm5`, `assessments/behavioral_observations`, `assessments/manuals`, `assessments/neurocognitive`) use descriptive filenames and are auto-ingested on RAG rebuild
- Future: `kb ask --corpus typst|quarto|...` dispatch for Tier 2 routing
- Future: FastAPI service mode so agents can call KB as a microservice
- Future: Citation rendering converting returned spans into clickable wikilink references in generated reports

## Related Concepts

- [[concepts/retrieval-augmented-generation]] — retrieval pattern underlying Tier 2
- [[concepts/vector-search]] — embedding-based search for large sub-corpora
- [[concepts/hybrid-search-retrieval]] — combined retrieval for Tier 2 sub-corpora
- [[concepts/local-first-architecture]] — privacy-driven deployment model
- [[concepts/phi-data-handling]] — handling of patient-identifiable information
- [[concepts/clinical-data-privacy]] — broader clinical data governance
- [[concepts/clinical-guidelines]] — professional practice guidelines housed in the shared reference layer
- [[concepts/behavioral-rating-scales]] — key content type in the shared references layer
- [[concepts/wais-iv]] — example assessment manual housed in the shared manuals directory
- [[concepts/per-patient-workspace]] — individualized workspaces built on the shared KB foundation
- [[concepts/luria-neuropsych-pipeline]] — the agent pipeline this KB serves
- [[concepts/neuropsychological-assessment-pipeline]] — clinical workflow context
- [[concepts/multi-agent-orchestration]] — how agents coordinate KB queries
- [[concepts/role-based-llm-routing]] — routing queries to the right corpus
- [[concepts/narrative-report-generation]] — downstream consumer of KB content
- [[concepts/validity-language]] — key content category in the wiki tier
- [[concepts/luria-overview]] — broader agent pipeline context
- [[concepts/dsm5-diagnosis-normalization]] — normalization of DSM-5 diagnostic codes used across the KB
- [[concepts/recommendation-rag-pipeline]] — pipeline supporting the recommendations shared reference directory

## Related Documents
- [[summaries/README]] — README for the recommendations/ shared reference directory
- [[summaries/README_luria]] — README for the Luria KB Tier 1 wiki

See also: [[summaries/2026-02-11-this-session-is-being-continued-from-a-previous-co]]

See also: [[summaries/AUTISM_RAG_SYSTEM_DOCUMENTATION]]

See also: [[summaries/DIAGNOSIS_FIX_SUMMARY]]

See also: [[summaries/SESSION_SUMMARY]]

See also: [[summaries/redesign_20260623110817]]

See also: [[summaries/redesign_20260623110910]]

See also: [[summaries/Luria_AI_Q4_Investor_Memo_2026]]