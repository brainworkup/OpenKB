---
doc_type: short
full_text: sources/cerner-autotext.md
---

# Cerner Autotext

Cerner Autotext is a Cerner EHR documentation feature that inserts pre-defined dynamic tokens into notes and reports, automatically resolving them to patient-specific data at render time. It is used to reduce manual entry, standardize note construction, and support efficient EHR documentation workflows.

## Core Idea

Autotext tokens are written in bracketed shortcut form and populated by the Cerner system when documentation is generated. This makes them useful for structured note authoring, especially where clinicians repeatedly combine standard phrasing with individualized patient data.

This document is particularly relevant to document templates and to specialty reporting contexts such as neuropsychology.

## Main Token Categories

### Patient Name Tokens
Used to insert patient name components:
- `0_st_auto_text_name_first`
- `0_st_auto_text_name_last`
- `0_st_auto_text_name_full`
- `0_st_auto_text_mr_mrs`

### Pronoun Tokens
Used to generate patient-appropriate pronouns:
- `0_st_auto_text_he_she`
- `0_st_auto_text_him_her`
- `0_st_auto_text_his_hers`
- Capitalized variants such as `0_st_auto_text_he_she_caps`

### Demographic Tokens
Used for common identifying and demographic information:
- `0_st_auto_text_mrn`
- `0_st_auto_text_gender`
- `0_st_auto_text_age`
- `0_st_auto_text_dob`
- `0_st_auto_text_age_dob`

### Date Tokens
Used to insert current or encounter dates:
- `0_st_auto_text_curdate`
- `0_st_auto_text_admit_dt`

### Care Team / Provider Tokens
Used to insert clinician and provider information:
- `0_st_careteam_md_epr`
- `0_st_careteam_referring_md`
- `0_st_careteam_attending_md`

The source also notes an example named admitting physician value: TRAMPUSH PH.D., JOEY W.

### Clinical Content Tokens
Used to pull structured patient content into notes:
- `[ * Medication List ]`
- `[ Allergies ]`
- `[ Social History ]`
- `[ Medical Problems ]`
- `[ * Diagnosis Related Orders ]`
- `[ Problems List ]`

The document identifies `[ Social History ]` as the preferred token for social history content and `[ Medical Problems ]` as the preferred token for diagnosis/problem-list content.

## Known Issue

A notable limitation is that the `[ Age ]` token is reported as non-functional. The recommended workaround is to use `[ Birth Date ]` instead, or related DOB-based tokens.

## Documentation Use

Cerner Autotext is framed as especially useful for structured clinical writing where reusable templates must still incorporate patient-specific details. This connects directly to:
- [[concepts/clinical-data-management]]
- [[concepts/clinical-report-structure]]
- [[concepts/neuropsychological-reporting]]

In neuropsychological and other assessment-based reporting, Autotext supports faster report assembly, greater consistency, and fewer transcription errors.

## Related Source Context

This page references [[summaries/cerner-autotext]] as the source context and also connects to report-writing practices discussed in Neuropsych Report Writing.

## Related Concepts
- [[concepts/modular-report-architecture]]
- [[concepts/neuropsychological-report-variables]]
- [[concepts/clinical-narrative-generation]]
- [[concepts/neuropsychological-assessment-workflow]]
- [[concepts/clinical-communication-register]]
