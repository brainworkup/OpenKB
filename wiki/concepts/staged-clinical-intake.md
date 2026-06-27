---
sources: [summaries/clinical-assessment.md, summaries/attention-problems.md, summaries/redesign_20260623110910.md, summaries/redesign_20260623110817.md, summaries/CARS2-Manual_extracted.md, summaries/multi_patient_transcript.md, summaries/nse_narrative.md]
brief: Structured multi-phase intake that organizes history before formal assessment.
---

# Staged Clinical Intake and History Gathering

Staged clinical intake is a structured, multi-phase approach to collecting patient history, background, and current concerns prior to formal neuropsychological testing. By organizing information into discrete stages — each with its own focus, data sources, and interpretive limits — clinicians ensure that the pre-testing narrative is comprehensive, internally consistent, privacy-conscious, and defensible as a clinical document.

This concept is central to the [[summaries/nse_narrative]] prompt template, which structures intake evidence into four sequential stages that feed directly into a [[concepts/neurobehavioral-status-exam]] summary. The [[summaries/multi_patient_transcript]] document is a concrete example of this process in action, illustrating how multi-informant, multi-modal data converge during intake.

The broader clinical importance of staging is reinforced by [[summaries/clinical-assessment]], which emphasizes that behavior and symptoms often vary across contexts and that single-source evaluation is frequently insufficient. In that sense, staged intake is not just an administrative sequence; it is an early clinical assessment framework for organizing discrepant, context-dependent evidence before standardized testing begins.

## Why Staged Intake Matters

Neuropsychological evaluations draw on heterogeneous information: old medical records, self-report questionnaires, structured interviews, rating scales, and behavioral observations collected across different times and contexts. Without a staging framework, this information risks being synthesized inconsistently, overinterpreted, or incompletely. A staged approach:

- Separates historical evidence from current presentation
- Distinguishes subjective patient report from clinician observation
- Creates a clear audit trail from raw inputs to synthesized narrative
- Supports [[concepts/phi-data-handling]] by isolating redaction and preprocessing as an explicit stage
- Accommodates multi-informant input across modalities such as in-person, phone, and telehealth
- Preserves the distinction between hypothesis generation during intake and confirmation through later testing
- Makes room for context-sensitive interpretation when reports diverge across home, school, work, or clinic settings

This last point is especially important in light of [[concepts/multi-informant-assessment]] and [[concepts/cross-informant-correspondence]]. As summarized in [[summaries/clinical-assessment]], informants often disagree in clinically meaningful ways rather than merely because of error. A staged intake provides the structure needed to capture those discrepancies cleanly before they are folded into broader diagnostic reasoning.

## The Four-Stage Model

### Stage 1 — Past Records Review
This stage consolidates longitudinal background: prior evaluation records, medical and psychiatric history, developmental and educational history, family history, prior interventions, and current medications. It anchors the current presentation within the patient's life history and prior care.

In [[summaries/multi_patient_transcript]], this stage is supplied largely by the parents, who describe a developmental arc spanning early childhood through adulthood. The history includes delayed language development, specialized preschool placement, and occupational and speech therapy. The absence of prior behavioral or emotional evaluations — despite significant early neurodevelopmental concerns — is itself clinically meaningful information captured at this stage.

Stage 1 is also where clinicians begin tracking onset and chronicity. As highlighted in [[summaries/clinical-assessment]], one important assessment question is whether symptoms are longstanding or newly emergent. This is particularly relevant for attention and executive concerns, where distinguishing developmental patterns from later-acquired change can shape the entire downstream evaluation.

### Stage 2 — Current Intake and History Materials
This stage captures the patient's current clinical picture through structured intake forms, rating scales, symptom checklists, and self-reported [[concepts/cognitive-domains]] concerns. It identifies functional impact areas that formal testing should clarify — bridging subjective complaint to objective inquiry.

In [[summaries/multi_patient_transcript]], current concerns include job loss following a pattern of task-switching, multiple car accidents linked to fatigue and poor judgment, social isolation accelerated by remote work, difficulty translating situational awareness into action, and restricted interests and food preferences. These concerns map directly onto domains that later testing may assess, including attention, executive functioning, language, social cognition, and adaptive functioning.

This stage is also where dimensional symptom description can be more useful than premature categorical labeling. [[summaries/clinical-assessment]] notes that dimensional approaches often support better cross-informant correspondence than categorical ones. In practice, Stage 2 benefits from describing severity, frequency, context, and functional impact rather than forcing early yes/no conclusions.

### Stage 3 — Tele-Intake Interview (and In-Person Collateral Interview)
A clinician-conducted interview adds a live, interactive layer. The transcript or summary captures the patient's narrative in their own words, while behavioral observations record clinician impressions of presentation, affect, language, cooperation, and social reciprocity. This stage is closely linked to [[concepts/clinical-communication-register]], as the clinician must calibrate language and rapport in real time.

[[summaries/multi_patient_transcript]] demonstrates how this stage can include real-time collateral informants — in this case, the father in person and the mother by phone. The clinician uses a semi-structured autism rating instrument during the interview itself, working through domains such as social-emotional understanding, emotional expression and regulation, relating to people, body use, adaptation to change and restricted interests, visual and auditory response, and sensory sensitivities. This is a distinctive feature: the structured instrument is embedded within the interview rather than administered separately, allowing the clinician to probe and contextualize ratings collaboratively with informants.

Key behavioral observations noted during the session include flat affect and monotone speech, limited spontaneous emotional expression, compulsive apologizing during tennis, stiffness with physical contact, and eye contact that requires deliberate effort especially when formulating responses. These observations emerge naturally in conversation and are therefore especially valuable for connecting reported symptoms to observed presentation.

From the standpoint of [[concepts/multi-informant-assessment]], Stage 3 is often where converging and diverging reports become most visible. A staged model helps the clinician preserve discrepancies rather than smoothing them over too early. Low agreement across informants does not automatically imply invalid data; as summarized in [[summaries/clinical-assessment]], disagreement may reflect genuine context dependence, differing observational access, or methodological artifacts. The intake interview is where those possibilities first become clinically legible.

### Stage 4 — PHI-Safe Preprocessing
Before any AI-assisted synthesis, a dedicated preprocessing stage applies redaction and anonymization. This operationalizes [[concepts/pii-redaction-pipelines]] and supports compliance with privacy regulations. Redaction notes from this stage are passed explicitly into the prompt to document what information was modified or removed.

This stage is essential when intake material is later processed within automated reporting workflows. It separates protected raw inputs from downstream summarization and supports a reproducible chain of custody for sensitive information.

## Multi-Informant Intake as a Core Feature

The transcript illustrates a critical design principle: staged intake is not simply a sequential process applied to a single data source, but a framework for triangulating across informants. The patient, father, and mother each contribute distinct perspectives:

- The **patient** provides first-person phenomenology — his understanding of his motivations, emotional states, social preferences, and awareness of his differences.
- The **father** contributes longitudinal behavioral observation across diverse contexts, often noting discrepancies between the patient's self-report and observed behavior.
- The **mother** provides early developmental detail unavailable from other sources — including pre-verbal behavior, early therapy history, sensory anomalies in infancy, and family decision-making about evaluation and intervention.

Disagreements between informants are themselves clinically informative and become part of the intake record. The clinician's role is to synthesize these perspectives rather than simply arbitrate between them.

This aligns directly with [[concepts/cross-informant-correspondence]]. As emphasized in [[summaries/clinical-assessment]], discrepancies can reflect at least three broad possibilities: the patient behaves consistently across settings, behaves differently across settings, or appears discrepant because of differences in methods or raters. Staged intake creates the documentation structure needed to preserve these alternatives without collapsing them prematurely.

## Integration with Structured Rating Instruments

The transcript demonstrates how a semi-structured instrument — in this case a domain-by-domain autism severity rating scale covering social-emotional understanding, emotional expression, relating to people, body use, adaptation to change, visual and auditory response, and sensory processing — can be embedded within the intake conversation. The clinician reads domain descriptions and severity anchors aloud, then invites patient and informant responses collaboratively. This approach:

- Makes the rating transparent and participatory rather than clinician-only
- Allows informants to contest, qualify, or add nuance to ratings in real time
- Produces a richer record than a checklist completed independently
- Anchors the emerging diagnostic hypothesis in the patient's own life narrative

This connects to [[concepts/behavioral-rating-scales]] as a method and to [[concepts/neurodevelopmental-clinical-intake]] as the broader intake context.

More generally, [[summaries/clinical-assessment]] underscores that instrument choice affects interpretability. Structured scales and checklists do not replace clinical judgment; they standardize observation and support comparison across informants and contexts. Within staged intake, their best use is to sharpen questions, reveal discrepancies, and organize symptom dimensions that later testing can evaluate more objectively.

## Integration with Report Generation

The output of staged intake feeds directly into the pre-testing narrative section of the [[concepts/neuropsychological-reporting]] workflow. In [[summaries/nse_narrative]], all four stages are injected as structured variables into a single prompt, which then synthesizes them into a cohesive [[concepts/neurobehavioral-status-exam]] summary. This approach exemplifies [[concepts/narrative-report-generation]] driven by structured clinical evidence.

The staged model also connects to the broader modular report architecture pattern, where distinct report sections correspond to distinct data-gathering phases and each phase has clear scope boundaries. [[summaries/multi_patient_transcript]] would populate Stage 3 of this model, with the collateral phone interview contributing additional Stage 1 and Stage 2 content that might otherwise be captured only through written records.

The value of this structure is not merely technical. It supports clinical assessment by ensuring that downstream summaries preserve source distinctions: what came from records, what came from questionnaires, what emerged in interview, and what was directly observed. That separation improves interpretability and reduces the risk of mixing historical report, current complaint, and clinician inference into a single undifferentiated narrative.

## Relationship to Downstream Testing

A well-executed staged intake does not pre-interpret neuropsychological results — it frames the questions that testing should answer. In [[summaries/multi_patient_transcript]], the clinician explicitly defers formal diagnosis pending formal testing, treating the intake session as hypothesis-generating rather than conclusive. The NSE summary produced from staged intake identifies what [[concepts/cognitive-domains]] and functional areas remain to be clarified through objective assessment, preserving the integrity of the subsequent evaluation. This boundary between pre-testing narrative and test interpretation is a core principle of [[concepts/clinical-report-structure]].

This is especially important for concerns like attention, executive dysfunction, and social communication differences, where symptom meaning depends on onset, chronicity, context, and functional impact. [[summaries/clinical-assessment]] highlights that attention problems may be longstanding yet untreated until some triggering event prompts evaluation. A staged intake is where that timeline is first assembled: when symptoms began, where they appeared, whether they were ever treated, and how they changed over time.

The session closes with a plan to continue the following day with formal neuropsychological testing — a clean illustration of how staged intake terminates at the point of handing off to standardized assessment.

## Related Concepts

- [[concepts/neurobehavioral-status-exam]]
- [[concepts/neuropsychological-reporting]]
- [[concepts/neuropsychological-assessment-pipeline]]
- [[concepts/narrative-report-generation]]
- [[concepts/clinical-report-structure]]
- [[concepts/phi-data-handling]]
- [[concepts/pii-redaction-pipelines]]
- [[concepts/cognitive-domains]]
- [[concepts/clinical-communication-register]]
- [[concepts/autism-spectrum-disorder-clinical-features]]
- [[concepts/behavioral-rating-scales]]
- [[concepts/neurodevelopmental-clinical-intake]]
- [[concepts/speech-language-development-disorders]]
- [[concepts/executive-function-deficits]]
- [[concepts/multi-informant-assessment]]
- [[concepts/cross-informant-correspondence]]
- [[concepts/trauma-informed-clinical-assessment]]
- [[concepts/premorbid-vs-acquired-attention-difficulties]]
- [[summaries/nse_narrative]]
- [[summaries/neurobehav.prompt]]
- [[summaries/multi_patient_transcript]]
- [[summaries/clinical-assessment]]

See also: [[summaries/CARS2-Manual_extracted]]

See also: [[summaries/redesign_20260623110817]]

See also: [[summaries/redesign_20260623110910]]

See also: [[summaries/attention-problems]]