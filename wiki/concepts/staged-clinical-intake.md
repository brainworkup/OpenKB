---
sources: [summaries/redesign_20260623110910.md, summaries/redesign_20260623110817.md, summaries/CARS2-Manual_extracted.md, summaries/multi_patient_transcript.md, summaries/nse_narrative.md]
brief: Multi-phase framework for gathering patient history before formal neuropsychological testing begins.
---

# Staged Clinical Intake and History Gathering

Staged clinical intake is a structured, multi-phase approach to collecting patient history, background, and current concerns prior to formal neuropsychological testing. By organizing information into discrete stages — each with its own focus and data sources — clinicians ensure that the pre-testing narrative is comprehensive, internally consistent, and defensible as a clinical document.

This concept is central to the [[summaries/nse_narrative]] prompt template, which structures intake evidence into four sequential stages that feed directly into a [[concepts/neurobehavioral-status-exam]] summary. The `multi_patient_transcript` document — a transcribed intake session involving a patient, his father, his mother (by phone), and a clinician — is a concrete example of this process in action, illustrating how multi-informant, multi-modal data converge during intake.

## Why Staged Intake Matters

Neuropsychological evaluations draw on heterogeneous information: old medical records, self-report questionnaires, structured interviews, and behavioral observations collected across different times and contexts. Without a staging framework, this information risks being synthesized inconsistently or incompletely. A staged approach:

- Separates historical evidence from current presentation
- Distinguishes subjective patient report from clinician observation
- Creates a clear audit trail from raw inputs to synthesized narrative
- Supports [[concepts/phi-data-handling]] by isolating redaction and preprocessing as an explicit stage
- Accommodates multi-informant input (patient, family members, collateral contacts) across different modalities (in-person, phone, telehealth)

## The Four-Stage Model

### Stage 1 — Past Records Review
This stage consolidates longitudinal background: prior evaluation records, medical and psychiatric history, developmental and educational history, family history, and current medications. It anchors the current presentation within the patient's life history and prior care.

In the `multi_patient_transcript`, this stage is supplied largely by the parents, who describe a developmental arc spanning early childhood (non-verbal until approximately age four, attended a specialized speech and language preschool, received occupational and speech therapy) through adolescence and into adulthood. The absence of prior behavioral or emotional evaluations — despite significant early neurodevelopmental concerns — is itself clinically meaningful information captured at this stage.

### Stage 2 — Current Intake and History Materials
This stage captures the patient's current clinical picture through structured intake forms, rating scales, and self-reported [[concepts/cognitive-domains]] concerns. It also identifies functional impact areas that neuropsychological testing should clarify — bridging subjective complaint to objective inquiry.

In the transcript, current concerns include job loss following a pattern of task-switching that the patient attributed to an urge to reduce perceived disorder, multiple car accidents linked to fatigue and judgment lapses, social isolation accelerated by remote work, difficulties translating situational awareness into action (described vividly in the context of tennis), and restricted food preferences and social topics. These concerns map directly onto the domains that formal testing should address.

### Stage 3 — Tele-Intake Interview (and In-Person Collateral Interview)
A clinician-conducted interview adds a live, interactive layer. The transcript or summary captures the patient's narrative in their own words, while behavioral observations record clinician impressions of presentation, affect, language, and cooperation. This stage is closely linked to [[concepts/clinical-communication-register]], as the clinician must calibrate language and rapport in real time.

The `multi_patient_transcript` demonstrates how this stage can include real-time collateral informants — in this case, the father (in person) and the mother (by phone). The clinician uses a semi-structured autism rating instrument during the interview itself, working through domains such as social-emotional understanding, emotional expression and regulation, relating to people, body use, adaptation to change and restricted interests, visual and auditory response, and sensory sensitivities. This is a distinctive feature: the structured instrument is embedded within the interview rather than administered separately, allowing the clinician to probe and contextualize ratings collaboratively with informants.

Key behavioral observations noted during the session include flat affect and monotone speech, limited spontaneous emotional expression, compulsive apologizing during tennis, stiffness with physical contact, and eye contact that requires deliberate effort especially when formulating responses — all observations that emerge naturally in the flow of conversation rather than through formal testing.

### Stage 4 — PHI-Safe Preprocessing
Before any AI-assisted synthesis, a dedicated preprocessing stage applies redaction and anonymization. This operationalizes [[concepts/pii-redaction-pipelines]] and supports compliance with privacy regulations. Redaction notes from this stage are passed explicitly into the prompt to document what information was modified or removed.

## Multi-Informant Intake as a Core Feature

The transcript illustrates a critical design principle: staged intake is not simply a sequential process applied to a single data source, but a framework for triangulating across informants. The patient, father, and mother each contribute distinct perspectives:

- The **patient** provides first-person phenomenology — his own understanding of his motivations (e.g., wanting to reduce invoice "mess"), his emotional states, his social preferences, and his awareness of his differences.
- The **father** contributes longitudinal behavioral observation across diverse contexts (tennis, disc golf, family meals, road trips), often noting discrepancies between the patient's self-report and observed behavior.
- The **mother** provides early developmental detail unavailable from other sources — including pre-verbal behavior, early therapy history, sensory anomalies in infancy, and the family's decision-making about evaluation and intervention.

Disagreements between informants (e.g., father and mother diverging on eye contact severity) are themselves clinically informative and become part of the intake record. The clinician's role is to synthesize these perspectives rather than arbitrate between them.

## Integration with Structured Rating Instruments

The transcript demonstrates how a semi-structured instrument — in this case a domain-by-domain autism severity rating scale covering social-emotional understanding, emotional expression, relating to people, body use, adaptation to change, visual and auditory response, and sensory processing — can be embedded within the intake conversation. The clinician reads domain descriptions and severity anchors aloud, then invites patient and informant responses collaboratively. This approach:

- Makes the rating transparent and participatory rather than clinician-only
- Allows informants to contest, qualify, or add nuance to ratings in real time
- Produces a richer record than a checklist completed independently
- Anchors the emerging diagnostic hypothesis (here, [[concepts/autism-spectrum-disorder-clinical-features]]) within the patient's own life narrative

This connects to [[concepts/behavioral-rating-scales]] as a method and to [[concepts/neurodevelopmental-clinical-intake]] as the broader intake context.

## Integration with Report Generation

The output of staged intake feeds directly into the pre-testing narrative section of the [[concepts/neuropsychological-reporting]] workflow. In the [[summaries/nse_narrative]] template, all four stages are injected as structured variables into a single prompt, which then synthesizes them into a cohesive [[concepts/neurobehavioral-status-exam]] summary. This approach exemplifies [[concepts/narrative-report-generation]] driven by structured clinical evidence.

The staged model also connects to the broader modular report architecture pattern, where distinct report sections correspond to distinct data-gathering phases, and each phase has clear scope boundaries. The `multi_patient_transcript` would populate Stage 3 of this model, with the collateral phone interview contributing additional Stage 1 and Stage 2 content that might otherwise be captured only through written records.

## Relationship to Downstream Testing

A well-executed staged intake does not pre-interpret neuropsychological results — it frames the questions that testing should answer. In the transcript, the clinician explicitly defers formal diagnosis pending formal testing, treating the intake session as hypothesis-generating rather than conclusive. The NSE summary produced from staged intake identifies what [[concepts/cognitive-domains]] and functional areas remain to be clarified through objective assessment, preserving the integrity of the subsequent evaluation. This boundary between pre-testing narrative and test interpretation is a core principle of [[concepts/clinical-report-structure]].

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
- [[summaries/nse_narrative]]
- [[summaries/neurobehav.prompt]]
- [[summaries/multi_patient_transcript]]

See also: [[summaries/CARS2-Manual_extracted]]

See also: [[summaries/redesign_20260623110817]]

See also: [[summaries/redesign_20260623110910]]