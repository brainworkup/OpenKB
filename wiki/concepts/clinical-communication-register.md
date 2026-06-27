---
sources: [summaries/cerner-autotext.md, summaries/attention-problems.md, summaries/LLM Benchmark Comparison.md, summaries/Apply-to-Y-Combinator-JWT.md, summaries/bwu.neuro.reports.recs.books.md, summaries/nse_narrative.md, summaries/neurocog.prompt.md, summaries/neurobehav.prompt.md, summaries/customization.md, summaries/multi_patient_transcript.md, summaries/clinical-validity-reviewer.md, summaries/brainworkup-branding-concepts.md, summaries/brainworkup-brand-voice-guide.md]
brief: How neuropsychology practitioners vary language formality, tone, and terminology across clinical audiences and contexts.
---

# Clinical Communication Register

Clinical communication register refers to the systematic variation in language style, formality, terminology, and emotional tone that skilled practitioners deploy across different professional contexts. In neuropsychology and healthcare more broadly, the same clinical findings must be communicated in fundamentally different ways depending on who is receiving the information and why.

The concept is central to [[summaries/brainworkup-brand-voice-guide]], which formalizes a multi-register communication system for a pediatric and forensic neuropsychology practice. The branding concepts explored in [[summaries/brainworkup-branding-concepts]] extend this idea further — demonstrating that register is not just a writing consideration but a whole-practice design decision, shaping visual identity, naming, typography, and website architecture alongside language.

## Why Register Matters

A single neuropsychological finding — say, a child scoring at the 21st percentile on working memory — must be expressed differently depending on the audience:

- **To a parent:** "Your child may need more support when following multi-step directions in the classroom."
- **To a referring physician:** "Working memory index at the 21st percentile, consistent with low average range; see attached report for recommendations."
- **In a forensic report:** "Working memory capacity, as measured by standardized instruments, yielded a standard score of 88, placing the examinee at the 21st percentile relative to age-matched normative samples."

The underlying data is identical. The register — vocabulary, sentence structure, hedging, warmth, formality — changes entirely.

## Register in Clinical Report Writing

One of the most demanding applications of register management in neuropsychology is the integrated narrative summary — the section of an evaluation report that synthesizes performance across neurocognitive domains. Two closely related prompt configurations encode specific register requirements into their generation instructions: the `neurobehav` prompt (see [[summaries/neurobehav.prompt]]) for behavioral and personality domains, and the `neurocog` prompt (see [[summaries/neurocog.prompt]]) for the full range of cognitive performance domains.

Both prompts share the same core register requirements:

- **Past tense throughout** — consistent with formal clinical convention, which treats assessment as a completed event
- **Third-person voice** — maintains the objective stance expected in professional reports
- **Mixed technical and accessible language, leaning professional** — the core register challenge: findings must be interpretable by another clinician reviewing the case *and* by a parent reading the report for the first time
- **Continuous paragraph-form prose** — avoids the fragmented, list-based register of informal notes or bullet-point summaries
- **Patient-specific pronouns** — a register sensitivity that personalizes the narrative without undermining formality

The `neurocog` prompt is particularly notable for the breadth of cognitive domains it must synthesize in a single integrated register: general cognitive ability, academic skills, verbal and language processing, visual perception and construction, memory, attention and executive functioning, motor and sensorimotor functioning, ADHD, social cognition, and emotional/behavioral and personality functioning. Producing a coherent narrative that moves across all ten domains — shifting from, say, a standard score interpretation in the intelligence domain to a qualitative observation about sensorimotor coordination — requires continuous register calibration sentence by sentence.

The model parameters chosen for both prompts reinforce the register goal: a low temperature (0.4) and moderate presence and frequency penalties suppress stylistic variability, producing consistent, clinically precise language rather than creative or expressive variation. This is register control implemented at the inference level. The high token ceiling (8192 in the `neurocog` prompt) accommodates the register demand of completeness — a sophisticated multi-domain summary cannot be truncated without sacrificing clinical credibility.

The behavioral and personality domain is particularly register-sensitive because it addresses findings that are emotionally loaded for families — diagnoses or patterns related to ADHD, autism spectrum disorder, personality functioning, and social cognition — while simultaneously requiring the precision and defensibility expected by referring physicians, school teams, and legal reviewers. See [[concepts/narrative-report-generation]] and [[concepts/neuropsychological-prompt-engineering]] for related technical approaches.

## Register as a Branding Dimension

[[summaries/brainworkup-branding-concepts]] makes explicit what the brand voice guide implies: that register must be encoded into the visual and structural identity of a practice, not just its written content. Each of the seven branding concepts maps to a distinct register posture:

- **Warm concepts** ("Growing Minds,

See also: [[summaries/nse_narrative]]

See also: [[summaries/multi_patient_transcript]]

See also: [[summaries/bwu.neuro.reports.recs.books]]

See also: [[summaries/Apply-to-Y-Combinator-JWT]]

See also: [[summaries/LLM Benchmark Comparison]]

See also: [[summaries/attention-problems]]

See also: [[summaries/cerner-autotext]]