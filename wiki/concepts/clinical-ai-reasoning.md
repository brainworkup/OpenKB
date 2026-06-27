---
sources: [summaries/Luria_AI_Q4_Investor_Memo_2026.md, summaries/LLM Benchmark Comparison.md, summaries/redesign_20260623110910.md, summaries/redesign_20260623110817.md]
brief: AI systems that perform structured, evidence-grounded clinical reasoning.
---

# Clinical AI Reasoning

Clinical AI reasoning refers to the use of AI systems — typically large language models — to perform structured, evidence-grounded reasoning over patient data in support of clinical decision-making. Unlike generic chatbots, clinical AI reasoning systems are designed to cite specific sources, surface contradictions, integrate heterogeneous clinical evidence, and produce provisional formulations that clinicians can inspect and challenge.

In the Luria context, clinical AI reasoning is not just report drafting. It is the reasoning layer that connects intake, score processing, interpretation, differential diagnosis support, and final narrative generation across the broader [[concepts/neuropsychological-assessment-workflow]]. The Q4 2026 investor memo reframes this capability as part of a production-ready neuropsychological AI platform with domain-specific reasoning chains across 8 neuropsychological domains, automated validity checks, and clinician-validated accuracy claims.

## Core Characteristics

### Evidence Grounding
Every claim made by the AI is tied to a numbered source citation. The clinician can trace any inference back to its origin — a test score, an interview excerpt, a prior record. This distinguishes clinical AI reasoning from unconstrained generative output and is essential for professional accountability.

### Multi-Source Integration
Clinical reasoning requires synthesizing across heterogeneous inputs: standardized test scores, behavioral rating scales, caregiver interviews, school records, and medical history. Effective clinical AI reasoning systems hold all of these in context simultaneously and surface convergent patterns.

### Provisional Impression with Uncertainty
Clinical AI reasoning produces **provisional** conclusions — clearly labeled hypotheses with recommended confirmatory steps, not final diagnoses. This mirrors the epistemics of human clinical reasoning: form the best hypothesis, flag what is pending, recommend what would resolve ambiguity.

### Clinician-in-the-Loop
The AI acts as a copilot or consultant, not an autonomous decision-maker. The clinician questions, challenges, and accepts or rejects AI-generated impressions. The Luria Console is described explicitly as "a clinical copilot you argue with."

### Differential Diagnosis Support
A key feature of clinical AI reasoning is not just summarizing findings, but discriminating among competing explanations. The Q4 2026 memo explicitly highlights an interpretation engine with differential diagnosis support, reinforcing that the system is meant to compare hypotheses, test dissociations, and preserve diagnostic specificity rather than merely generate plausible prose.

### Structured Validity Checking
Clinical AI reasoning is constrained by validity. The Q4 2026 memo adds that Luria automates score processing with clinical validity checks, indicating that reasoning begins only after data quality and interpretive constraints are assessed. This ties reasoning to the broader concerns captured in [[concepts/validity-and-response-styles]] and [[concepts/report-review-qa]].

## Implementation in Luria

Both [[summaries/redesign_20260623110817]] and [[summaries/redesign_20260623110910]] provide detailed product-level illustrations of clinical AI reasoning in neuropsychological practice, with the latter adding technical depth on infrastructure and a planned knowledge base layer. The Q4 2026 investor memo in [[summaries/Luria_AI_Q4_Investor_Memo_2026]] extends this picture by presenting the same reasoning capabilities as part of a production-ready workflow orchestration layer spanning intake → scoring → interpretation → reporting, with quality assurance automation and clinician feedback loops.

### The Console & Synthesis Panel
The Console (Page 05 of [[summaries/redesign_20260623110910]]) is the primary clinical reasoning interface. The clinician poses a diagnostic question in natural language:

> *"Is the inattention primary ADHD, or secondary to the dysgraphia and academic frustration?"*

Luria responds with structured reasoning organized around evidence types:
- **Cross-setting evidence** — Conners-4 parent and teacher forms both elevated (T=72/70)
- **Dissociation analysis** — Attention/Executive composite (SS 76) depressed independently of graphomotor speed
- **Historical evidence** — onset predates formal writing demands per caregiver interview

Each point is numbered and mapped to a source in the Evidence Rail. The response concludes with a **Provisional Impression** and recommended confirmatory tests (CPT-3, Beery VMI). Actions available to the clinician include [Add to Synthesis], [Draft this section], and [Show reasoning] — keeping the clinician in control of how AI output enters the formal record.

The Q4 memo clarifies that this sort of interaction is backed by domain-specific reasoning chains and an interpretation engine intended to scale across multiple neuropsychological domains rather than functioning as a single-purpose prompt.

### Live Synthesis Panel
The Report Workspace (Page 01) includes a live synthesis panel that continuously updates a plain-language clinical impression as new scores are entered. This is a lightweight form of clinical AI reasoning operating as ambient background processing rather than on-demand consultation. In the demo case ("Biggie Smalls," age 7), the panel narrates: *"Two findings converge on the referral question. Attention/Executive (SS 76) and graphomotor output are both depressed, while verbal reasoning is intact — a profile consistent with ADHD-Inattentive with co-occurring dysgraphia rather than a global delay."*

This continuous synthesis aligns with the Q4 memo’s framing of an end-to-end orchestration system in which reasoning is embedded throughout the workflow rather than isolated to a final drafting step.

### Pattern Detection in the Cognitive Map
The Amber-theme Cognitive Map (Page 07) uses AI to detect domain clusters that co-vary across 13 neurocognitive domains displayed as a visual constellation (node size = test breadth, color = severity). In the demo case, it identifies the **frontal-graphomotor cluster** — Attention, Executive, and Motor domains depressed together while Verbal Reasoning is intact — and names the pattern as the ADHD-Inattentive + dysgraphia signature. Convergence is scored (4/4) and spared strengths are called out explicitly (Verbal 102, Reasoning 96). This is visual, gestalt-level clinical reasoning.

The investor memo’s mention of reasoning chains across 8 neuropsychological domains suggests a more formalized domain architecture underlying this kind of pattern detection.

### AI-Assisted Intake Structuring
Clinical AI reasoning extends upstream into the Patient Intake module (Page 02). The intake assistant auto-structures the referral question into discrete clinical concerns, flags missing information (e.g., absence of prior psychoeducational testing), and surfaces suggested follow-up items. This represents clinical AI reasoning applied not to test scores but to the unstructured language of caregiver reports and referral letters.

This upstream structuring fits with the memo’s positioning of Luria as a full clinical workflow platform rather than only a report generator.

### Document-Grounded Fact Extraction
The Clinical Office (Page 04) uses a RAG pipeline (24 chunks embedded) to extract discrete, source-linked facts from ingested records: *"Expressive speech delay treated age 3-4 ↳ Medical Records · p.3."* These grounded facts pre-fill the report background and serve as evidence available to later reasoning steps in the Console.

### Report Drafting with Citation Awareness
The Report Builder (Page 08) generates narrative prose grounded in the same evidence used during the Console reasoning phase. The AI is described as "drafting from 6 tests & 3 sources · citations on," meaning the reasoning layer and the writing layer share the same evidence substrate. Voice, reading level, and section insertions are clinician-controlled.

The Q4 memo reinforces this linkage by describing a single workflow that unifies scoring, interpretation, and reporting with quality assurance automation. In that framing, clinical AI reasoning is the bridge between raw clinical inputs and final [[concepts/narrative-report-generation]].

## Relationship to RAG and Knowledge Architecture

Clinical AI reasoning depends on reliable retrieval of patient-specific evidence. In Luria, this is implemented through a [[concepts/retrieval-augmented-generation]] pipeline (Clinical Office, Page 04) that indexes source documents into embedded chunks and extracts discrete facts linked to source pages. The reasoning system queries this indexed evidence rather than relying on model memory.

Page 10 of [[summaries/redesign_20260623110910]] describes a planned integration of OpenKB (Open Knowledge Base) as a backend knowledge layer — not yet implemented in the app. Rather than per-query RAG that rediscovers knowledge from scratch, OpenKB compiles clinical knowledge once into a persistent wiki using LLMs, with cross-references pre-existing and contradictions flagged. This would extend clinical AI reasoning from patient-specific retrieval to persistent, accumulating domain knowledge. See [[concepts/knowledge-base-architecture]] for the architectural principles.

The document handling distinction is notable: short documents are read in full by the LLM, while long PDFs are indexed into a hierarchical tree that the LLM reads instead of the full text — enabling better retrieval from lengthy clinical documents without context overflow.

The investor memo adds a practical production lens: Luria’s knowledge base is described as part of the technical infrastructure for continuous learning, suggesting that compiled knowledge is not merely an architectural idea but part of the platform’s scaling strategy.

## PHI Safety as a Reasoning Constraint

Clinical AI reasoning over real patient data requires strict privacy controls. In Luria's architecture (Page 09 of [[summaries/redesign_20260623110910]]), the `redactPhi()` guard intercepts all data before it reaches any LLM, and the [[concepts/local-llm-inference]] fallback chain ensures PHI-sensitive reasoning stays on-device:

1. **oMLX** (local, PHI-safe) — primary
2. **vMLX** (local, PHI-safe) — fallback
3. **Ollama** (local, PHI-safe) — fallback
4. **Cloud** (non-PHI only) — blocked for PHI requests

The `restrictToPreferredProviders: true` flag enforces the local-only gate. `AsyncLocalStorage` threads an abort signal through the generate pipeline so a job can be cancelled mid-fallback.

The Q4 2026 memo strengthens this section by explicitly describing the reasoning stack as **privacy-preserving**, **HIPAA-aligned**, and **local-first**, with a multi-model architecture that combines local and cloud LLMs for quality/cost optimization while preserving privacy constraints. This architecture relates to broader [[concepts/phi-data-handling]], [[concepts/clinical-data-privacy]], and [[concepts/phi-deidentification-pipeline]] concerns.

See also [[concepts/privacy-first-software]] and [[concepts/local-first-architecture]] for the architectural principles that enable safe clinical AI reasoning.

## Reasoning Quality and Transparency

Key quality properties for clinical AI reasoning systems:

- **Source transparency** — every inference cites its evidence
- **Provisional framing** — outputs are hypotheses, not conclusions
- **Dissociation analysis** — reasoning explicitly addresses what is *not* impaired, not just what is
- **Cross-setting validation** — findings in a single context are flagged as less reliable than cross-setting convergence
- **Pending flags** — the system identifies what information is still needed before finalizing an impression
- **Convergence scoring** — the system quantifies how many independent evidence streams agree (e.g., 4/4 convergence in the Cognitive Map)
- **Validity-aware interpretation** — conclusions are constrained by score validity and response-style checks
- **Diagnostic specificity preservation** — automation should not collapse nuanced distinctions into generic labels
- **QA integration** — reasoning outputs can be checked through automated and clinician review loops

The Q4 2026 memo adds early outcome claims that matter for this concept page: 94% clinician-validated accuracy, higher reported accuracy than manual junior clinician work (94% vs 87%), consistent documentation quality, and no degradation in diagnostic specificity. These are not definitive proof of reasoning quality, but they show how Luria is operationalizing evaluation criteria for clinical AI reasoning in production-like settings. They also connect naturally to [[concepts/llm-evaluation]], [[concepts/neuropsychological-score-interpretation]], and [[concepts/report-review-qa]].

## Relationship to Clinical Workflow Automation

Clinical AI reasoning is one layer within a broader automation stack. The investor memo makes this especially clear by framing the system as an end-to-end platform that automates case intake, score processing, interpretation, and reporting while keeping clinicians in supervisory roles. In this architecture, reasoning is the central interpretive function that turns extracted facts and computed scores into clinically meaningful hypotheses.

This makes clinical AI reasoning especially relevant to:
- [[concepts/neuropsychological-assessment-automation]]
- [[concepts/neuropsychological-assessment-pipeline]]
- [[concepts/clinical-ai-copilot]]
- [[concepts/human-in-the-loop-clinical-ai]]
- [[concepts/healthcare-workforce-automation]]

The memo’s business framing also highlights why reasoning quality matters operationally: if reasoning is reliable, clinician time savings can scale with case volume without sacrificing specificity.

## Related Concepts

- [[concepts/neuropsychological-synthesis]] — the clinical practice of integrating multi-domain findings into a coherent formulation
- [[concepts/narrative-report-generation]] — translating AI reasoning into formal clinical prose
- [[concepts/neuropsychological-assessment-workflow]] — the workflow context in which clinical AI reasoning is embedded
- [[concepts/retrieval-augmented-generation]] — the retrieval mechanism that grounds AI claims in patient records
- [[concepts/local-llm-inference]] — privacy-safe inference infrastructure for clinical settings
- [[concepts/phi-data-handling]] — handling of protected health information during AI reasoning
- [[concepts/luria-overview]] — overview of the Luria platform where clinical AI reasoning is implemented
- [[concepts/luria-neuropsych-pipeline]] — the end-to-end pipeline that clinical AI reasoning supports
- [[concepts/multi-agent-orchestration]] — how multiple specialized agents coordinate to produce clinical reasoning outputs
- [[concepts/fallback-strategy]] — provider fallback logic that keeps clinical reasoning available and PHI-safe
- [[concepts/knowledge-base-architecture]] — persistent compiled knowledge layer that may underpin future clinical AI reasoning
- [[concepts/neurodevelopmental-clinical-intake]] — the intake phase where clinical AI reasoning begins
- [[concepts/cognitive-domains]] — the domain structure over which clinical reasoning operates
- [[concepts/llm-provider-abstraction]] — the abstraction layer enabling provider-agnostic clinical inference
- [[concepts/clinical-ai-copilot]] — the clinician-facing interaction model for argument-driven AI assistance
- [[concepts/human-in-the-loop-clinical-ai]] — the supervisory model that keeps clinicians responsible for final decisions
- [[concepts/healthcare-workforce-automation]] — the broader operational context in which clinical AI reasoning creates leverage

See also: [[summaries/LLM Benchmark Comparison]] and [[summaries/Luria_AI_Q4_Investor_Memo_2026]]