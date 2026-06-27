---
sources: [summaries/clinical-assessment.md, summaries/cerner-autotext.md, summaries/attention-problems.md, summaries/Luria_AI_Q4_Investor_Memo_2026.md, summaries/Apply-to-Y-Combinator-JWT.md, summaries/2026-06-26-2133-plan.md, summaries/redesign_20260623110910.md]
brief: End-to-end process from intake to synthesis and neuropsychological reporting.
---

# Neuropsychological Assessment Workflow

The neuropsychological assessment workflow is the end-to-end clinical process by which a practitioner moves from a referral question through structured data collection, record review, cognitive testing, synthesis, and final report generation. In modern software implementations such as Luria, this workflow is operationalized as a local-first, AI-assisted pipeline designed to preserve clinical quality, integrate across neurocognitive and neurobehavioral domains, and keep sensitive patient data under strict privacy controls. The workflow is especially valuable in settings where scoring, interpretation, and report writing are the main bottlenecks, including pediatric, hospital-based, and forensic neuropsychology.

A central clinical constraint is that neuropsychological assessment cannot rely on test scores or a single observer alone. The broader logic of [[summaries/clinical-assessment]] emphasizes that cognitive, neurological, and psychological functioning must often be evaluated across settings and informants, using standardized instruments, structured checklists, and context-sensitive interpretation. This is especially important when behavior varies across home, school, interview, and formal testing contexts.

Luria is framed not simply as a documentation tool, but as an agent-assisted clinical platform intended to support or automate the full evaluation process from intake through final report. As described in [[summaries/Apply-to-Y-Combinator-JWT]], [[summaries/redesign_20260623110910]], and [[summaries/Luria_AI_Q4_Investor_Memo_2026]], the core design premise is that neuropsychological work requires more than isolated score interpretation: it requires cross-domain integration, structured clinical reasoning, preservation of clinical voice, and privacy-aware handling of PHI. The Q4 2026 investor memo strengthens this framing by positioning the workflow itself as Luria's core product surface: an end-to-end system for case intake, score processing, interpretation, quality assurance, and final reporting.

---

## The Five-Stage Model

The canonical workflow comprises five sequential phases, each building on the last:

1. **Intake** — Structured history capture: demographics, referral question, developmental and medical history, family history, and presenting concerns. Data is captured once and reused across downstream stages.
2. **Documents** — Ingestion and indexing of external records such as medical notes, school records, and prior evaluations. Grounded facts are extracted for later synthesis.
3. **Cognitive Map** — Administration, scoring, validation, and interpretation of standardized tests across neurocognitive domains such as reasoning, memory, attention/executive abilities, language, and academic skills.
4. **Synthesis** — AI-assisted clinical reasoning that integrates test scores, behavioral ratings, observations, interview content, and historical records into a coherent formulation.
5. **Report** — Structured narrative generation that produces the final clinical deliverable in a configurable voice and format.

This staged model reflects both clinical logic and software architecture. In practice, it turns a traditionally manual, hours-long reporting process into a reproducible pipeline while preserving the clinician's role in judgment, review, feedback, and sign-off. The Q4 2026 memo describes this as an end-to-end case intake → scoring → interpretation → reporting workflow, with quality assurance automation layered throughout. See also [[concepts/neuropsychological-assessment-pipeline]] and [[concepts/luria-neuropsych-pipeline]].

---

## Stage Details

### Intake

The [[concepts/neurodevelopmental-clinical-intake]] stage transforms a free-text referral into structured, tagged concerns. In Luria-style workflows, an intake assistant can:
- auto-structure the referral into discrete concerns such as inattention, handwriting difficulty, mood symptoms, or rule-out diagnoses
- flag missing historical data
- identify follow-up questions relevant to differential diagnosis
- organize developmental, educational, and medical history for later reuse

Demographic and contextual fields such as date of birth, grade, handedness, primary language, referral source, and setting feed directly into [[concepts/neuropsychological-report-variables]].

This stage matters because the referral question frames the entire downstream interpretation process. In neuropsychology, a good workflow does not merely collect facts; it organizes them around the eventual clinical question and report narrative. In production-oriented systems, intake also becomes the first point where workflow orchestration and data completeness checks can reduce downstream rework.

The clinical-assessment synthesis adds an important refinement: intake should also anticipate where symptoms may differ by context and where single-source reporting may be misleading. For common referral problems such as attention concerns, intake should capture onset, chronicity, prior treatment, context of first recognition, and whether symptoms appear longstanding or newly acquired. This is especially relevant to [[concepts/premorbid-vs-acquired-attention-difficulties]] and [[concepts/developmental-vs-acquired-cognitive-symptoms]].

### Document Ingestion

Records are converted from PDF and related formats into analyzable text, then indexed using [[concepts/retrieval-augmented-generation]]. Facts are linked to source material so later report statements can remain grounded in the record. This stage corresponds closely to [[concepts/clinical-data-management]] and depends on strong [[concepts/phi-data-handling]] safeguards.

Supported source types often include:
- clinical interview transcripts
- prior medical records
- school records and IEPs
- prior psychoeducational or neuropsychological evaluations
- rating scale summaries

This stage is especially important because neuropsychological interpretation rarely depends on test scores alone. Historical context, teacher reports, caregiver observations, prior diagnoses, and treatment history often determine whether a pattern reflects a primary disorder, a secondary effect, or a contextual stressor. See also [[concepts/pdf-data-extraction]], [[concepts/report-ingestion-pipeline]], and [[concepts/neuropsych-report-parsing]].

The clinical-assessment material further underscores that structured checklists and standardized instruments complement document review rather than replace it. Record ingestion is most useful when it preserves context-specific details that later explain informant discrepancies, treatment timing, and whether a symptom pattern was recognized only after a triggering event.

### Cognitive Map

The Cognitive Map stage visualizes and interprets scores across multiple [[concepts/cognitive-domains]]. Rather than treating each test in isolation, the workflow groups findings into clinically meaningful patterns across domains such as attention, memory, language, executive control, visuospatial skills, processing speed, and academic achievement.

AI-assisted pattern detection can identify clusters of co-varying weaknesses and strengths, helping the clinician move from raw scores to interpretable patterns. For example:

> *Frontal-graphomotor cluster: attention, executive control, and motor output co-vary and are jointly depressed while verbal reasoning is relatively spared.*

Domain status categories are typically simplified into labels such as Average, Borderline, and Clinical Concern, while retaining access to the underlying score structure. In the Q4 2026 memo, this stage is expanded beyond simple score interpretation to include score processing automation with clinical validity checks and domain-specific reasoning chains across eight neuropsychological domains. That description highlights a key workflow principle: scoring is not just arithmetic; it is a clinically constrained transformation step that benefits from built-in validity review before interpretation proceeds. See also [[concepts/neuropsychological-score-interpretation]], [[concepts/neuropsychological-test-scores]], [[concepts/neuropsychological-tests]], and [[concepts/executive-function-deficits]].

The clinical-assessment synthesis strengthens this stage by emphasizing the range of core domains typically assessed: memory, language, visual-spatial skills, executive functions, and attention. It also highlights that dimensional measures may sometimes produce better cross-informant correspondence than categorical approaches, which matters when integrating test results with rating scales and observational data.

### Synthesis

The Synthesis stage is the core interpretive layer and aligns with [[concepts/clinical-ai-copilot]], [[concepts/clinical-ai-reasoning]], and [[concepts/neuropsychological-synthesis]]. Here, the system reasons across multiple evidence sources to support formulation rather than merely summarize content.

Typical functions include:
- integrating test scores with rating scales, history, and observations
- distinguishing primary from secondary diagnoses
- weighing cross-setting consistency
- identifying dissociations and converging evidence
- drafting a provisional formulation with grounded support

This stage reflects a central claim in Luria's positioning: the hard problem in neuropsychology is not producing generic boilerplate, but integrating across neurocognitive and neurobehavioral domains in a clinically credible voice. That integration is a distinctive feature of the field and a major reason generic cloud AI tools often underperform in this setting. The investor memo sharpens this claim further by emphasizing structured clinical reasoning, differential-diagnosis support, and clinician feedback loops as core workflow capabilities rather than optional enhancements. It also frames this interpretive layer as one source of measurable quality gains, citing clinician-validated accuracy and improved consistency relative to junior manual work. See [[summaries/Apply-to-Y-Combinator-JWT]] and [[summaries/Luria_AI_Q4_Investor_Memo_2026]].

The new clinical-assessment synthesis adds a major interpretive principle: disagreement across informants is often clinically meaningful rather than mere noise. Findings summarized from [[summaries/clinical-assessment]] draw on [[concepts/cross-informant-correspondence]] and [[concepts/multi-informant-assessment]] to show that parent, teacher, clinician, and self-report agreement is often modest, especially for internalizing symptoms. Within a workflow, that means synthesis should explicitly test whether discrepancies reflect:
- true cross-context variability
- differences in symptom visibility across settings
- measurement artifacts or rater effects
- differences in instrument type or thresholding

This makes multi-informant interpretation a core workflow task rather than an optional add-on. In practical terms, the synthesis layer should preserve source attribution, compare converging versus diverging reports, and avoid over-collapsing context-specific presentations into a single undifferentiated summary.

### Report Builder

The [[concepts/narrative-report-generation]] stage assembles the final report from standard components such as:
1. Reason for Referral
2. Background and History
3. Tests Administered
4. Results by Domain
5. Summary and Formulation
6. Recommendations

AI can draft each section from structured variables, source documents, and interpreted scores, with clinician controls for tone, audience, and modular inserts. This aligns with [[concepts/clinical-report-structure]], [[concepts/modular-report-architecture]], [[concepts/clinical-narrative-generation]], and [[concepts/neuropsychological-reporting]].

The report is the primary product delivered to patients, families, schools, attorneys, and referring providers. In practice, this is often the most labor-intensive part of the evaluation and can dominate total case time. A well-designed workflow therefore treats report generation not as an afterthought, but as the culmination of all prior stages. The Q4 2026 memo reinforces this point by framing the system's value around large per-case time savings, comprehensive report generation, and quality assurance automation without reported loss of diagnostic specificity.

The clinical-assessment synthesis suggests an additional reporting standard: final narratives should preserve clinically meaningful context, especially when symptoms differ by setting or when treatment history shows delayed recognition. For example, attention problems may be longstanding yet only formally addressed after injury, illness, school failure, or structured evaluation. Report generation should therefore support longitudinal phrasing, explicit discussion of onset and chronicity, and careful distinction between developmental and acquired explanations.

---

## Privacy and Local Inference

A defining constraint of the workflow is that all protected health information must remain local or be handled under strict controls. This is not only a technical preference but a clinical and operational requirement. In the Luria framing, local-first design is a key differentiator because neuropsychological evaluations often contain highly sensitive developmental, psychiatric, educational, and forensic information.

The [[concepts/local-first-architecture]] is enforced through measures such as:
- PHI guards on data sent to language models
- provider routing that prefers local inference
- deidentification or redaction steps where appropriate
- defaults that block cloud processing for PHI-sensitive tasks

See [[concepts/local-llm-inference]], [[concepts/phi-deidentification-pipeline]], [[concepts/llm-provider-abstraction]], [[concepts/clinical-data-privacy]], and [[concepts/privacy-first-software]].

The investor memo adds a more deployment-oriented articulation of this same principle: privacy-preserving architecture, local-first processing, and a multi-model stack that combines local and cloud LLMs for quality and cost optimization while remaining HIPAA-aligned. This suggests that in practice the workflow may selectively route subtasks across models, but privacy constraints still govern architecture at the workflow level. This local-first requirement also helps explain why the neuropsychological assessment workflow differs from generic note-generation products: preserving privacy, traceability, and clinical voice is inseparable from the workflow itself.

The clinical-assessment synthesis reinforces this by linking modern assessment work to clinical NLP and local AI use in sensitive documentation environments. In workflow terms, privacy is not isolated to inference infrastructure; it shapes ingestion, synthesis, source tracking, and report drafting across the full pipeline.

---

## Agent Pipeline

Under the hood, the workflow can be orchestrated as an agent pipeline, with specialized components handling intake, extraction, interpretation, synthesis, and drafting. This maps to [[concepts/multi-agent-orchestration]], [[concepts/subagent-architecture]], [[concepts/agent-pipeline-state-management]], and [[concepts/langgraph-agent-workflows]].

In Luria, the broader vision is an agent-based system that can execute most of the workflow from start to finish with coordinated agents and subagents. This includes collecting data from varied sources, organizing it into reusable structure, reasoning across evidence, and generating a clinically usable report. The goal is not just section-by-section text generation, but end-to-end workflow execution.

The Q4 2026 memo makes this architecture more operational by describing workflow orchestration, real-time clinician feedback loops, and quality assurance automation as production features rather than purely conceptual design goals. It also points to knowledge-base support for continuous learning and infrastructure integrations intended to fit clinician workflows more seamlessly. This architecture is particularly well suited to neuropsychology because the workflow is naturally staged, evidence-heavy, and modular, while still requiring a final synthesis layer that preserves clinician oversight.

The clinical-assessment material suggests another useful agent design principle: specialized components should track not only extracted facts and scores, but also source type, informant identity, context of observation, and degree of convergence across sources. This helps preserve clinically relevant distinctions during synthesis and reduces the risk of flattening conflicting evidence into generic summaries.

---

## Sample Case: Biggie Smalls (Age 7)

The Luria redesign illustrates the full workflow with a concrete case:
- **Referral:** Inattention and illegible handwriting; teacher reports bright but falling behind
- **Intake flags:** Mild expressive speech delay at age 3; paternal ADHD history
- **Test battery:** WISC-V, academic measures, memory testing, attention/executive tasks, and rating scales
- **Pattern:** Frontal-graphomotor cluster with relatively spared verbal reasoning
- **Provisional impression:** ADHD-Inattentive with co-occurring [[concepts/dysgraphia]]
- **Recommended next tests:** targeted measures for graphomotor function and sustained attention

This example shows how the workflow moves from complaint to grounded formulation by integrating history, observed functioning, test scores, and domain-level pattern analysis. It also illustrates why Luria emphasizes full-workflow support: the clinical value emerges from the sequence of stages and their reuse of structured data, not from any single report-writing step in isolation.

Viewed through the lens of [[summaries/clinical-assessment]], this case also shows why multi-informant interpretation matters. Teacher concerns, developmental history, test performance, and rating scales may each reflect different aspects of the child's functioning. The workflow is strongest when it can represent agreement, disagreement, and context dependence directly rather than forcing premature consensus.

---

## Why This Workflow Matters

The workflow addresses a longstanding problem in neuropsychology: evaluations are data-rich but scoring, interpretation, and report production remain highly manual. A straightforward case may take many hours, while pediatric and [[concepts/forensic-neuropsychological-evaluation]] cases can take substantially longer. Much of that burden comes from synthesizing across domains and producing a report that is clinically accurate, readable, and useful for real-world decision-making.

The modern neuropsychological assessment workflow therefore serves several purposes at once:
- reducing repetitive manual reporting work
- improving consistency and reuse of structured data
- preserving source-grounded interpretation
- supporting cross-domain synthesis
- integrating standardized instruments, checklists, and observational evidence
- handling multi-informant discrepancies as interpretable clinical data
- maintaining strict privacy controls
- helping clinicians scale without sacrificing quality

The Q4 2026 investor memo adds an important business and systems perspective to this concept. It argues that workflow automation is not just an efficiency feature but a response to neuropsychologist shortages, documentation inconsistency, and burnout. In that framing, the workflow becomes a mechanism for [[concepts/healthcare-workforce-automation]] while still preserving the clinician's supervisory role. The memo's reported gains in case throughput, time reduction, and clinician-validated accuracy support the claim that neuropsychological assessment workflow design can be both a clinical quality intervention and a scalability strategy.

The clinical-assessment synthesis broadens that claim by clarifying what quality means in practice: not merely faster scoring or cleaner prose, but better handling of context, chronicity, source discrepancies, and instrument choice. In this sense, the workflow is simultaneously a clinical model, a software model, and an operational model for expanding neuropsychological capacity.

---

## Related Concepts

- [[concepts/neuropsychological-assessment-automation]]
- [[concepts/neuropsychological-assessment-pipeline]]
- [[concepts/neuropsychological-reporting]]
- [[concepts/staged-clinical-intake]]
- [[concepts/clinical-narrative-generation]]
- [[concepts/behavioral-rating-scales]]
- [[concepts/neuropsychological-tests]]
- [[concepts/luria-neuropsych-pipeline]]
- [[concepts/luria-overview]]
- [[concepts/knowledge-base-architecture]]
- [[concepts/clinical-ai-copilot]]
- [[concepts/clinical-data-management]]
- [[concepts/clinical-data-privacy]]
- [[concepts/narrative-report-generation]]
- [[concepts/local-first-architecture]]
- [[concepts/healthcare-ai-regulation]]
- [[concepts/multi-informant-assessment]]
- [[concepts/cross-informant-correspondence]]
- [[concepts/cognitive-domains]]
- [[concepts/premorbid-vs-acquired-attention-difficulties]]
- [[concepts/developmental-vs-acquired-cognitive-symptoms]]

See also: [[summaries/2026-06-26-2133-plan]], [[summaries/redesign_20260623110910]], [[summaries/Apply-to-Y-Combinator-JWT]], [[summaries/Luria_AI_Q4_Investor_Memo_2026]]

See also: [[summaries/attention-problems]]

See also: [[summaries/cerner-autotext]]

See also: [[summaries/clinical-assessment]]