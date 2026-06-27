---
sources: [summaries/clinical-assessment.md, summaries/attention-problems.md, summaries/2026-06-26-2133-plan.md, summaries/multi_patient_transcript.md, summaries/sirf_synthesis.md, summaries/nse_narrative.md]
brief: Structured pre-testing narrative summarizing referral context, history, and observations.
---

# Neurobehavioral Status Exam (NSE)

The Neurobehavioral Status Exam (NSE) — also referred to in current documentation as **Neurobehavioral Exam (STT)** following a terminology rebranding effort — is a structured clinician-authored narrative that forms the pre-testing portion of a comprehensive [[concepts/neuropsychological-reporting]] report. It synthesizes available intake information — historical records, patient-reported concerns, interview observations, behavioral ratings, and administrative preprocessing notes — into a cohesive clinical summary that contextualizes the purpose and scope of the evaluation before objective testing occurs.

Within the broader frame of [[summaries/clinical-assessment]], the NSE functions as a core clinical assessment document: it organizes the evidence needed to understand cognitive, neurological, and psychological concerns before formal testing and helps ensure that later interpretation is anchored in context rather than isolated test scores.

## Terminology Note

The feature name has been updated from "Status Exam" (and variants such as "Neurobehavioral Status Exam") to **Neurobehavioral Exam (STT)** to maintain consistency with changes already implemented in the codebase. Documentation files across `docs/` and `notes/` directories are being updated to reflect this rebranding. See [[concepts/terminology-rebranding]] for the broader pattern of keeping documentation aligned with codebase naming conventions. The canonical term going forward is **Neurobehavioral Exam (STT)**, though legacy references to "NSE" or "Status Exam" may still appear in older files and summaries.

## Clinical Role

The NSE serves as the foundational narrative layer of a neuropsychological evaluation. It documents why a patient was referred, what their background reveals, how concerns present across contexts, and what domains the subsequent formal testing should address. Critically, the NSE does **not** include diagnoses, differential diagnoses, treatment recommendations, or interpretation of objective test scores. It is a pre-interpretive document — a synthesis of context, not a statement of conclusions.

This scope limitation preserves the appropriate boundary between the intake/history phase and the testing/interpretation phase of a neuropsychological evaluation, supporting professional standards and [[concepts/clinical-report-structure]].

The NSE is also a key place where clinicians begin distinguishing developmental patterns from acquired change. In attention-related referrals, for example, the narrative may document whether concentration problems appear longstanding and consistent with neurodevelopmental history or instead seem to follow a later neurological event. This distinction is central to [[concepts/developmental-vs-acquired-cognitive-symptoms]] and helps frame later interpretation without prematurely reaching conclusions.

The clinical-assessment synthesis also reinforces that single-source evaluation is often insufficient, especially with children and adolescents whose symptoms may vary across home, school, and clinical settings. Accordingly, the NSE often acts as the first integrated narrative space where multi-informant evidence is organized and where discrepancies are treated as clinically meaningful rather than automatically dismissed as noise.

## Staged Evidence Model

As described in [[summaries/nse_narrative]], the NSE prompt template organizes input into four sequential stages:

### Stage 1 — Past Records Review
This stage incorporates prior medical, psychiatric, psychological, developmental, educational, and family history, along with current medications. It provides longitudinal context for understanding the patient's neurobehavioral profile.

This stage is especially important when symptoms such as inattention, disorganization, or working-memory complaints may predate the referral event by many years. Historical evidence can help establish whether attention problems are longstanding rather than newly acquired, which is often essential to the later neuropsychological formulation.

The broader clinical-assessment literature summarized in [[summaries/clinical-assessment]] underscores the importance of onset and chronicity here: historical review may reveal that attention problems were present for years but not formally treated until a triggering event prompted evaluation. That temporal structure can be highly important for later differential thinking, even though the NSE itself remains pre-interpretive.

### Stage 2 — Current Intake and History Materials
This stage covers summaries from intake forms, behavioral rating scales, and structured history-taking. It also captures subjective cognitive complaints and functional impact areas that testing should clarify. See [[concepts/behavioral-rating-scales]] for more on rating instrument use in this context.

Within this stage, the NSE often records how attentional symptoms affect academic, occupational, or daily functioning. It can also preserve clinically relevant nuances such as a history of untreated concentration problems that only came to formal attention after a triggering event, decline, or evaluation referral.

This stage is also where the logic of [[concepts/multi-informant-assessment]] becomes especially relevant. Intake materials may include parent, teacher, patient, or other collateral perspectives that do not fully agree. Consistent with [[concepts/cross-informant-correspondence]], the NSE should preserve these differences in a clinically organized way rather than collapsing them into an oversimplified single account.

### Stage 3 — Tele-Intake Interview
This stage integrates the clinician's observations from a remote or in-person clinical interview, including behavioral observations and a summary of the patient's self-reported concerns. Behavioral observations documented here contribute directly to the NSE narrative voice. See [[concepts/staged-clinical-intake]] for the broader intake framework.

The interview stage is often where the clinician clarifies symptom timeline and trajectory. For attention complaints, this may include whether the patient describes lifelong distractibility, school-based difficulties, or chronic inefficiency versus a more recent onset associated with injury, illness, or other neurological change. The NSE can therefore capture the provisional historical distinction between longstanding and acquired attention difficulties while reserving definitive interpretation for later sections.

The interview is also one of the main places where contextual variation becomes visible. A patient may describe different functioning at work, home, or school, and collateral sources may describe a different pattern still. The NSE should document these patterns clearly because divergence across settings may reflect genuine context sensitivity rather than mere measurement error.

### Stage 4 — PHI-Safe Preprocessing
Before the NSE is drafted, patient identifying information undergoes redaction or preprocessing review to ensure compliance with privacy regulations. This stage connects to [[concepts/phi-data-handling]] and [[concepts/pii-redaction-pipelines]].

## Output Characteristics

A well-formed NSE summary:
- Is written in professional, accessible past-tense prose
- Uses flowing paragraphs (no bullet points)
- Describes referral context, salient history, behavioral observations, and current concerns
- Integrates relevant multi-source information while preserving important context differences
- Identifies what later testing should explore without pre-interpreting those results
- Avoids all diagnostic, differential, or treatment-planning language

When attention is a referral concern, a strong NSE will describe the reported pattern with appropriate temporal framing — for example, noting whether problems appear chronic, developmentally rooted, or potentially acquired — and will specify which cognitive domains later testing should examine, such as attention regulation, executive functioning, and [[concepts/working-memory]].

More broadly, the NSE helps set up later evaluation of core [[concepts/cognitive-domains]] including attention, memory, language, visual-spatial skills, and executive functions, while staying strictly on the descriptive side of the assessment process.

This aligns with broader principles of [[concepts/clinical-communication-register]] and [[concepts/narrative-report-generation]].

## Documentation Consistency

The rebranding from "Status Exam" to "Neurobehavioral Exam (STT)" affects several key documentation files, including `docs/DATAFLOW.md`, `docs/INTEGRATION.md`, `docs/SCORE_EXTRACTION.md`, `notes/redesign.md`, and `notes/luria_app_redesign.qmd`. The update ensures that the distinction between the feature name and the content type is preserved — for example, "Neurobehavioral Exam (STT) transcript" is used where context requires explicit reference to the content type. This is part of a broader commitment to documentation-as-code principles, ensuring that human-facing docs remain synchronized with the evolving codebase.

## Automation and Prompt Engineering

The NSE is a strong candidate for LLM-assisted generation due to its structured inputs and constrained output scope. The prompt template in [[summaries/nse_narrative]] demonstrates how [[concepts/neuropsychological-prompt-engineering]] can be applied: a role-prompted LLM is given staged, templated inputs and asked to produce a clinician-facing narrative that respects strict content boundaries.

This approach connects to the broader automated reporting pipeline described in [[concepts/neuropsychological-assessment-automation]] and [[concepts/neuropsychological-assessment-pipeline]], where structured prompt configuration drives consistent report section generation. The use of template variables (`{patient_name}`, `{referral_reason}`, etc.) reflects patterns discussed in [[concepts/neuropsychological-report-variables]] and [[concepts/neuropsychological-prompt-configuration]].

Attention-related history illustrates why structured prompting matters: the generated NSE must preserve chronology, distinguish self-reported longstanding symptoms from possible acquired decline, represent cross-context or cross-informant differences accurately, and flag domains for follow-up testing without drifting into diagnosis or recommendation language. This makes the NSE a good example of controlled clinical narrative generation rather than freeform summarization.

The clinical-assessment synthesis also suggests a quality criterion for automated NSE generation: systems should preserve context-specific distinctions, avoid flattening discrepant reports, and support later probabilistic or evidence-based interpretation without performing that interpretation prematurely.

## Related Concepts

- [[concepts/neuropsychological-reporting]]
- [[concepts/clinical-report-structure]]
- [[concepts/staged-clinical-intake]]
- [[concepts/narrative-report-generation]]
- [[concepts/neuropsychological-prompt-engineering]]
- [[concepts/neuropsychological-prompt-configuration]]
- [[concepts/neuropsychological-report-variables]]
- [[concepts/neuropsychological-assessment-automation]]
- [[concepts/neuropsychological-assessment-pipeline]]
- [[concepts/behavioral-rating-scales]]
- [[concepts/phi-data-handling]]
- [[concepts/pii-redaction-pipelines]]
- [[concepts/clinical-communication-register]]
- [[concepts/cognitive-domains]]
- [[concepts/terminology-rebranding]]
- [[concepts/developmental-vs-acquired-cognitive-symptoms]]
- [[concepts/premorbid-vs-acquired-attention-difficulties]]
- [[concepts/executive-function-deficits]]
- [[concepts/working-memory]]
- [[concepts/multi-informant-assessment]]
- [[concepts/cross-informant-correspondence]]
- [[concepts/neurobehavioral-status-exam]]

See also: [[summaries/sirf_synthesis]]

See also: [[summaries/multi_patient_transcript]]

See also: [[summaries/2026-06-26-2133-plan]]

See also: [[summaries/attention-problems]]

See also: [[summaries/clinical-assessment]]