---
sources: [summaries/2026-06-26-2133-plan.md, summaries/multi_patient_transcript.md, summaries/sirf_synthesis.md, summaries/nse_narrative.md]
brief: Structured clinician narrative forming the pre-testing context section of a neuropsychological evaluation report.
---

# Neurobehavioral Status Exam (NSE)

The Neurobehavioral Status Exam (NSE) — also referred to in current documentation as **Neurobehavioral Exam (STT)** following a terminology rebranding effort — is a structured clinician-authored narrative that forms the pre-testing portion of a comprehensive [[concepts/neuropsychological-reporting]] report. It synthesizes all available intake information — historical records, patient-reported concerns, interview observations, and administrative preprocessing notes — into a cohesive clinical summary that contextualizes the purpose and scope of the evaluation before any objective testing occurs.

## Terminology Note

The feature name has been updated from "Status Exam" (and variants such as "Neurobehavioral Status Exam") to **Neurobehavioral Exam (STT)** to maintain consistency with changes already implemented in the codebase. Documentation files across `docs/` and `notes/` directories are being updated to reflect this rebranding. See [[concepts/terminology-rebranding]] for the broader pattern of keeping documentation aligned with codebase naming conventions. The canonical term going forward is **Neurobehavioral Exam (STT)**, though legacy references to "NSE" or "Status Exam" may still appear in older files and summaries.

## Clinical Role

The NSE serves as the foundational narrative layer of a neuropsychological evaluation. It documents why a patient was referred, what their background reveals, and what domains the subsequent formal testing should address. Critically, the NSE does **not** include diagnoses, differential diagnoses, treatment recommendations, or interpretation of objective test scores. It is a pre-interpretive document — a synthesis of context, not a statement of conclusions.

This scope limitation preserves the appropriate boundary between the intake/history phase and the testing/interpretation phase of a neuropsychological evaluation, supporting professional standards and [[concepts/clinical-report-structure]].

## Staged Evidence Model

As described in [[summaries/nse_narrative]], the NSE prompt template organizes input into four sequential stages:

### Stage 1 — Past Records Review
This stage incorporates prior medical, psychiatric, psychological, developmental, educational, and family history, along with current medications. It provides longitudinal context for understanding the patient's neurobehavioral profile.

### Stage 2 — Current Intake and History Materials
This stage covers summaries from intake forms, behavioral rating scales, and structured history-taking. It also captures subjective cognitive complaints and functional impact areas that testing should clarify. See [[concepts/behavioral-rating-scales]] for more on rating instrument use in this context.

### Stage 3 — Tele-Intake Interview
This stage integrates the clinician's observations from a remote or in-person clinical interview, including behavioral observations and a summary of the patient's self-reported concerns. Behavioral observations documented here contribute directly to the NSE narrative voice. See [[concepts/staged-clinical-intake]] for the broader intake framework.

### Stage 4 — PHI-Safe Preprocessing
Before the NSE is drafted, patient identifying information undergoes redaction or preprocessing review to ensure compliance with privacy regulations. This stage connects to [[concepts/phi-data-handling]] and [[concepts/pii-redaction-pipelines]].

## Output Characteristics

A well-formed NSE summary:
- Is written in professional, accessible past-tense prose
- Uses flowing paragraphs (no bullet points)
- Describes referral context, salient history, behavioral observations, and current concerns
- Identifies what later testing should explore without pre-interpreting those results
- Avoids all diagnostic, differential, or treatment-planning language

This aligns with broader principles of [[concepts/clinical-communication-register]] and [[concepts/narrative-report-generation]].

## Documentation Consistency

The rebranding from "Status Exam" to "Neurobehavioral Exam (STT)" affects several key documentation files, including `docs/DATAFLOW.md`, `docs/INTEGRATION.md`, `docs/SCORE_EXTRACTION.md`, `notes/redesign.md`, and `notes/luria_app_redesign.qmd`. The update ensures that the distinction between the feature name and the content type is preserved — for example, "Neurobehavioral Exam (STT) transcript" is used where context requires explicit reference to the content type. This is part of a broader commitment to documentation-as-code principles, ensuring that human-facing docs remain synchronized with the evolving codebase.

## Automation and Prompt Engineering

The NSE is a strong candidate for LLM-assisted generation due to its structured inputs and constrained output scope. The prompt template in [[summaries/nse_narrative]] demonstrates how [[concepts/neuropsychological-prompt-engineering]] can be applied: a role-prompted LLM is given staged, templated inputs and asked to produce a clinician-facing narrative that respects strict content boundaries.

This approach connects to the broader automated reporting pipeline described in [[concepts/neuropsychological-assessment-automation]] and [[concepts/neuropsychological-assessment-pipeline]], where structured prompt configuration drives consistent report section generation. The use of template variables (`{patient_name}`, `{referral_reason}`, etc.) reflects patterns discussed in [[concepts/neuropsychological-report-variables]] and [[concepts/neuropsychological-prompt-configuration]].

## Related Concepts

- [[concepts/neuropsychological-reporting]]
- [[concepts/clinical-report-structure]]
- [[concepts/staged-clinical-intake]]
- [[concepts/narrative-report-generation]]
- [[concepts/neuropsychological-prompt-engineering]]
- [[concepts/neuropsychological-prompt-configuration]]
- [[concepts/neuropsychological-report-variables]]
- [[concepts/neuropsychological-assessment-automation]]
- [[concepts/behavioral-rating-scales]]
- [[concepts/phi-data-handling]]
- [[concepts/pii-redaction-pipelines]]
- [[concepts/clinical-communication-register]]
- [[concepts/cognitive-domains]]
- [[concepts/terminology-rebranding]]

See also: [[summaries/sirf_synthesis]]

See also: [[summaries/multi_patient_transcript]]

See also: [[summaries/2026-06-26-2133-plan]]