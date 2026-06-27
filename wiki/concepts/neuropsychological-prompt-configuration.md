---
sources: [summaries/nt_interpretation.md, summaries/neurocog.prompt.md]
brief: LLM configuration parameters and persona settings for clinical neuropsychological report generation.
---

# Neuropsychological Prompt Configuration

Neuropsychological prompt configuration refers to the structured set of model parameters, system personas, and output instructions used to direct a large language model (LLM) toward producing clinically appropriate neuropsychological narrative summaries. This approach treats the prompt itself as a clinical instrument — carefully tuned to balance technical precision, professional register, and interpretive sophistication.

See [[summaries/neurocog.prompt]] for the primary source document.

## Core Components

### Model Parameters

The configuration layer specifies inference-time hyperparameters that shape the model's generative behavior:

- **Temperature (0.4):** Low setting encourages deterministic, precise clinical language over creative variation.
- **Max Tokens (8192):** High ceiling accommodates comprehensive, multi-domain neuropsychological write-ups.
- **Top-P (1.0):** Full nucleus sampling range, allowing the low temperature to govern output diversity.
- **Presence Penalty (0.5):** Discourages topic repetition across a multi-domain report.
- **Frequency Penalty (0.5):** Reduces word-level repetition, supporting varied clinical prose.
- **Mirostat (0.8):** Adaptive perplexity control for consistent narrative quality throughout long outputs.
- **Stop Sequence (`--END--`):** Explicit generation boundary marker.

These parameters are discussed further in [[concepts/yaml-configuration]] and [[concepts/neuropsychological-prompt-engineering]].

### System Persona

The system prompt establishes a detailed clinical persona: a board-certified clinical neuropsychologist with expertise in psychodiagnostic assessment, neurodevelopment, attention disorders, executive functioning, autism spectrum disorder (ASD), and social-emotional development. This persona grounding is essential for [[concepts/clinical-communication-register]] — ensuring outputs conform to the conventions of formal neuropsychological reporting.

### Neurocognitive Domain Scope

The configuration references a ten-domain test battery:

1. General Cognitive Ability / Intelligence
2. Academic Skills
3. Verbal / Language Processing
4. Visual Perception / Construction
5. Memory
6. Attention / Executive Functioning
7. Motor / Sensorimotor
8. ADHD
9. Social Cognition
10. Emotional, Behavioral, and Personality Functioning

This domain structure maps directly to standard neuropsychological frameworks. See [[concepts/cognitive-domains]] and [[concepts/neuropsychological-tests]] for broader coverage of these domains.

### Output Directives

The prompt encodes explicit stylistic and structural instructions:

- **Third-person, past-tense voice** — consistent with formal clinical report conventions (see [[concepts/clinical-report-structure]])
- **Dual-audience language** — mixing technical terminology with accessible prose (see [[concepts/dual-audience-design]])
- **Integrated paragraph form** — narrative synthesis rather than fragmented bullet lists (see [[concepts/narrative-report-generation]])
- **Pronoun-sensitive writing** — patient pronouns are honored when specified

## Relationship to the Broader Pipeline

This configuration pattern sits at the intersection of several architectural concerns:

- **[[concepts/neuropsychological-reporting]]** — The output format targets formal clinical documentation.
- **[[concepts/neuropsychological-assessment-pipeline]]** — This prompt is one node in a larger automated assessment workflow.
- **[[concepts/neuropsychological-assessment-automation]]** — Prompt engineering replaces or augments manual interpretation steps.
- **[[concepts/modular-report-architecture]]** — Domain-specific prompts like this one compose into larger report structures.
- **[[concepts/local-llm-inference]]** — The configuration is designed to run on locally hosted models, supporting [[concepts/clinical-data-privacy]].
- **[[concepts/role-based-llm-routing]]** — Different prompt configurations handle different neurocognitive domains or clinical roles.

## Related Prompts and Summaries

- [[summaries/neurobehav.prompt]] — Companion prompt focused on behavioral and emotional interpretation domains.
- [[summaries/neuropsych-narrative-writer]] — Agent responsible for consuming these configurations and producing report text.
- [[summaries/cognition.instructions]] — Broader cognitive interpretation instructions.
- [[summaries/clinical-validity-reviewer]] — Downstream QA step for generated clinical narratives.

## Design Rationale

The careful calibration of inference parameters alongside a richly specified clinical persona reflects a core principle: **prompt configuration is itself a form of clinical instrument design**. Just as a neuropsychologist selects and standardizes test administration procedures, the prompt engineer must standardize the generative environment to produce reliable, valid, and defensible clinical language. Low temperature, explicit stop sequences, and domain-scoped system personas collectively reduce variance and increase the clinical utility of LLM-generated neuropsychological summaries.


See also: [[summaries/nt_interpretation]]