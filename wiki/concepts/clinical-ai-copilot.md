---
sources: [summaries/README_20260414001057.md, summaries/Luria_AI_Q4_Investor_Memo_2026.md, summaries/LLM Benchmark Comparison.md, summaries/redesign_20260623110910.md]
brief: An AI assistant embedded in clinical workflows that reasons over patient data and grounds every claim in source evidence.
---

# Clinical AI Copilot

A **clinical AI copilot** is an AI assistant deeply integrated into a clinician's workflow that reasons about patient data, generates hypotheses, drafts clinical text, and cites every claim back to a grounded source document. Unlike generic LLM chat interfaces, a clinical AI copilot is context-aware (it knows the active case, the test battery, and the document record), PHI-safe, and designed to support — not replace — clinical judgment.

The concept is illustrated in full in [[summaries/redesign_20260623110910]], which documents the redesign of Luria by brainworkup™.

---

## Core Characteristics

### 1. Case-Grounded Reasoning
The copilot does not answer in generalities. Every response is anchored to the specific patient's data:
- Test scores from the active battery
- Facts extracted from ingested source documents
- Structured intake history

In Luria's Console, each AI response is labeled with the number of sources cited (e.g., *"reasoned · 4 sources"*), and an **evidence rail** lists each cited document with score values and page references.

### 2. Dialectical Interaction
The interface is described as *"a clinical copilot you argue with."* The clinician poses a diagnostic question and the AI responds with structured reasoning, which the clinician can accept, challenge, or redirect. This mirrors a clinical consultation rather than a search query.

**Example exchange from Luria:**
> **DR:** Is the inattention primary ADHD, or secondary to dysgraphia and academic frustration?
>
> **Luria:** Three lines of evidence favor a primary attention disorder:
> - Cross-setting: Conners-4 elevated on both parent and teacher forms (T=72/70)
> - Dissociation from output: Attention/Executive SS 76 depressed independently of graphomotor speed
> - History: Onset predates formal writing demands per caregiver interview

### 3. Actionable Outputs
Each copilot response surfaces concrete next steps:
- [Add to Synthesis]
- [Draft this section]
- [Show reasoning]

This connects the reasoning loop to the [[concepts/neuropsychological-assessment-workflow]] and [[concepts/narrative-report-generation]] pipeline without requiring the clinician to re-enter data.

### 4. PHI Safety
A clinical AI copilot operates under strict data privacy constraints. In Luria, this is enforced by:
- `redactPhi()` PHI guard before any LLM call
- `restrictToPreferredProviders: true` keeping inference local (oMLX → vMLX → Ollama)
- Cloud fallback blocked for any PHI-containing request

See [[concepts/clinical-data-privacy]], [[concepts/phi-data-handling]], and [[concepts/local-llm-inference]] for related architecture patterns.

---

## Architectural Components

### Evidence Rail
A persistent sidebar listing every source cited in the current reasoning session, each linked to its document and score:
- Conners-4 T72: Inattention elevated parent T=72, teacher T=70
- NEPSY-II SS76: Attention/Executive composite 5th %ile
- Clinical Interview: verbatim quote with onset timing

This pattern relates to [[concepts/retrieval-augmented-generation]] and [[concepts/clinical-ai-reasoning]].

### Live Synthesis Panel
The Report Workspace maintains a continuously updated synthesis panel that narrates the emerging clinical pattern as new test scores are entered. This is distinct from the Console — it is always-on, not query-driven.

### Intake Assistant
On the Patient Intake screen, a secondary copilot surface:
- Auto-structures free-text referral questions into discrete clinical tags
- Flags missing data (e.g., no prior psychoeducational testing)
- Suggests follow-up questions

This relates to [[concepts/neurodevelopmental-clinical-intake]] and [[concepts/staged-clinical-intake]].

### Report Builder Integration
The copilot connects to the [[concepts/modular-report-architecture]] and [[concepts/clinical-narrative-generation]] pipeline:
- Drafts report sections from 6 tests & 3 source documents
- Respects voice setting (Clinical / Balanced / Parent-facing)
- Adjusts reading level on demand
- Inserts score tables, domain figures, and DSM-5 criteria blocks

---

## Relationship to the Cognitive Map

The Cognitive Map module (13 neurocognitive domains as a visual constellation) feeds directly into the copilot's reasoning. The AI detects clusters of co-varying depressed domains and names the pattern:

> **PATTERN DETECTED — Frontal-graphomotor cluster:** Attention, executive control, and motor output co-vary and are jointly depressed while verbal reasoning is spared. That dissociation is the classic ADHD-Inattentive + dysgraphia signature.

This connects to [[concepts/cognitive-domains]], [[concepts/neuropsychological-synthesis]], and [[concepts/executive-function-deficits]].

---

## Relationship to Knowledge Base Architecture

Page 10 of [[summaries/redesign_20260623110910]] describes a planned backend integration with OpenKB — a compiled wiki system that persists clinical knowledge across documents rather than re-deriving it on each query. This addresses a fundamental limitation of standard RAG approaches. See [[concepts/knowledge-base-architecture]] and [[concepts/knowledge-continuity]].

---

## Related Concepts

- [[concepts/clinical-ai-reasoning]] — how the AI structures and presents evidence chains
- [[concepts/neuropsychological-assessment-workflow]] — the full workflow the copilot operates within
- [[concepts/local-llm-inference]] — PHI-safe inference infrastructure
- [[concepts/llm-provider-abstraction]] — fallback chain and provider routing
- [[concepts/narrative-report-generation]] — drafting clinical text from structured data
- [[concepts/phi-deidentification-pipeline]] — redaction before any LLM call
- [[concepts/multi-agent-orchestration]] — section agents and orchestration layer
- [[concepts/luria-overview]] — the broader Luria platform context
- [[concepts/luria-neuropsych-pipeline]] — the full data pipeline
- [[concepts/retrieval-augmented-generation]] — document grounding mechanism
- [[concepts/neuropsychological-synthesis]] — the synthesis step the copilot supports


See also: [[summaries/LLM Benchmark Comparison]]

See also: [[summaries/Luria_AI_Q4_Investor_Memo_2026]]

See also: [[summaries/README_20260414001057]]