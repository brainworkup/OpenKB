---
sources: [summaries/clinical-assessment.md, summaries/attention-problems.md, summaries/redesign_20260623110910.md, summaries/redesign_20260623110817.md, summaries/bwu.neuro.reports.recs.building-verbal-skills.md, summaries/bwu.neuro.reports.recs.build-math-skills.md, summaries/CARS2-Manual_extracted.md, summaries/sirf_synthesis.md, summaries/nt_interpretation.md, summaries/neurocog.prompt.md, summaries/neurobehav.prompt.md, summaries/LLM_INTEGRATION.md, summaries/LLM_AGENT_MAP.md, summaries/CLAUDE.md, summaries/multi_patient_transcript.md, summaries/report_body.md, summaries/NP-20240415-001_report.md, summaries/README.md, summaries/neuropsych-narrative-writer.md, summaries/neuropsych-data-extractor.md, summaries/brainworkup-brand-voice-guide.md, summaries/template-system.md, summaries/SKILL.md, summaries/AGENTS_luria.md]
brief: How neuropsychology organizes and interprets major areas of cognition.
---

# Cognitive Domains in Neuropsychological Assessment

Cognitive domains are the standardized categories of mental function evaluated during neuropsychological assessments. Each domain maps to distinct neural systems and behavioral capacities. Identifying which domains are affected — and to what degree — is the core interpretive task of clinical neuropsychology. Domain-based interpretation is not only descriptive; it is also developmental and causal. In practice, clinicians must determine whether weakness in a given domain reflects a longstanding neurodevelopmental pattern, a newly acquired change, or both. This distinction is especially important for attention-related findings, where apparent impairment may represent either lifelong vulnerability or decline after neurological illness or injury.

## Canonical Domain Taxonomy

The [[summaries/AGENTS_luria]] extraction schema enumerates seven primary cognitive domains used as a classification framework:

| Domain | Core Abilities Captured |
|---|---|
| **Memory** | Encoding, storage, and retrieval of verbal and visual information |
| **Executive Function** | Planning, inhibition, cognitive flexibility, abstract reasoning |
| **Attention** | Sustained, selective, divided, and alternating attention |
| **Language** | Naming, fluency, comprehension, repetition |
| **Processing Speed** | Rate of cognitive operations under timed conditions |
| **Visuospatial** | Construction, perception, spatial reasoning |
| **Social Cognition** | Emotion recognition, theory of mind, social judgment |

This taxonomy aligns broadly with DSM-5 neurocognitive domain classifications and is widely used in both clinical and research contexts. The Luria toolkit (see [[summaries/README_luria]] and [[summaries/README]]) operationalizes this taxonomy directly in its analysis layer: the `luria analyze` command accepts domain names (e.g., `memory`, `attention`) as arguments, and the `compute_descriptive_stats` API takes a `domains` list drawn from these canonical labels.

## Extended Domain Taxonomy for Report Generation

The [[summaries/neuropsych-narrative-writer]] agent — stage 3 of the [[concepts/luria-neuropsych-pipeline]] — operates against a more granular domain taxonomy that maps directly to Quarto include file prefixes. This extended set covers all domains that appear in full neuropsychological reports:

| Prefix | Domain |
|---|---|
| `_02-01_iq` | General Cognitive Ability / IQ |
| `_02-02_academics` | Academic / Achievement |
| `_02-03_verbal` | Verbal / Language |
| `_02-04_spatial` | Visuospatial / Visual-Construction |
| `_02-05_memory` | Memory & Learning |
| `_02-06_executive` | Executive Function |
| `_02-07_motor` | Sensorimotor |
| `_02-08_social` | Social Cognition |
| `_02-09_adhd` | ADHD (multi-rater: self, parent, teacher variants) |
| `_02-10_emotion` | Emotion/Behavior (multi-rater) |
| `_02-11_adaptive` | Adaptive Functioning |
| `_03-00_sirf` | Summary, Impressions, Recommendations & Formulation |
| `_03-01_recs` | Recommendations |

This prefix schema is stable by design — `_quarto.yml` and `template.qmd` in the cingulate pipeline depend on the fixed ordering and must not be renumbered casually.

## Clinical Neuropsychologist Domain Framework

The [[summaries/neurocog.prompt]] prompt configuration documents the ten-domain battery framework used by a board-certified clinical neuropsychologist. This framework extends the seven-domain Luria taxonomy to include domains critical for comprehensive neurodevelopmental and neuropsychiatric evaluation:

1. **General Cognitive Ability / Intelligence** — Overall intellectual functioning, often anchored by composite IQ scores
2. **Academic Skills** — Reading, writing, mathematics, and applied academic achievement
3. **Verbal / Language Processing** — Receptive and expressive language, naming, fluency, and discourse
4. **Visual Perception / Construction** — Visuospatial reasoning, visual-motor integration, constructional praxis
5. **Memory** — Verbal and visual encoding, consolidation, retrieval, and recognition
6. **Attention / Executive Functioning** — Sustained attention, working memory, inhibition, set-shifting, planning
7. **Motor / Sensorimotor** — Fine and gross motor skills, sensory-perceptual functions, psychomotor speed
8. **ADHD** — Inattention, hyperactivity, impulsivity across multi-rater perspectives (self, parent, teacher)
9. **Social Cognition** — Emotion recognition, perspective-taking, pragmatic understanding, theory of mind
10. **Emotional, Behavioral, and Personality Functioning** — Mood, anxiety, conduct, personality traits, and psychopathology

This ten-domain structure is particularly important for evaluations targeting neurodevelopmental conditions such as ADHD and autism spectrum disorder. It is also useful when a clinician must separate pre-existing developmental weaknesses from acquired dysfunction, especially within attention and executive systems. The [[summaries/neurocog.prompt]] configuration pairs this domain framework with model parameters calibrated for clinical precision: low temperature (0.4) for consistent technical language, high token ceiling (8192) for comprehensive multi-domain narratives, and presence/frequency penalties to reduce redundancy across domain sections. The output register blends technical and lay language — accessible to both professionals and non-professional stakeholders — as described in [[concepts/clinical-communication-register]] and [[concepts/dual-audience-design]].

The companion [[summaries/neurobehav.prompt]] extends coverage into behavioral and emotional domains with dedicated prompting for behavioral rating scale interpretation.

## Domain Profiles in Clinical Practice: The Luria Redesign

The Luria.app redesign (see [[summaries/redesign_20260623110910]]) provides a concrete illustration of how cognitive domains are surfaced in a modern clinical interface. The Report Workspace and Cognitive Map modules organize all findings by domain, displaying scores, severity indicators, and AI-generated pattern narratives in real time.

### Neurocognitive Scores Panel (Report Workspace)

In the redesigned interface, the Neurocognitive panel presents evaluated domains in a structured table with standard scores, percentile ranks, and status classifications:

| Domain | Score | Status | Instrument |
|---|---|---|---|
| General Cognitive Ability | 96 | EVALUATED (Average) | WISC-V · 39th %ile |
| Academic Skills | 78 | CLINICAL CONCERN | WIAT-4 Written Expression · 7th %ile |
| Memory | 88 | BORDERLINE | WRAML-3 · 21st %ile |
| Attention / Executive | 76 | CLINICAL CONCERN | NEPSY-II · 5th %ile |

The Neurobehavioral panel adds a separate tier for multi-rater behavioral domains:
- **ADHD / Executive Function:** T72 (ELEVATED) — Conners-4 · Inattention · Very Elevated
- **Social Cognition:** PENDING — Awaiting administration

This two-tier organization — Neurocognitive and Neurobehavioral — mirrors the ten-domain battery framework above, separating performance-based cognitive testing from behavioral rating scale data. The interface tracks domain completion across 13 total domains, and both a progress tracker (4/7 neurocognitive evaluated) and a mobile companion view expose this progress to the clinician at a glance.

### Cognitive Map: The Constellation View (Amber Theme)

The Cognitive Map module (accessible via `⌘6`) renders all 13 domains as a visual constellation in which node size reflects test breadth and color encodes severity. For the example case (Biggie Smalls, age 7), the map surfaces:

| Domain | Score |
|---|---|
| Verbal / Language | 102 |
| General Cognitive | 96 |
| Memory | 88 |
| Academic Skills | 78 |
| Attention / Executive | 76 |
| Motor / Graphomotor | 72 |
| ADHD / Executive | 72 |

The AI pattern detection layer reads the constellation and names the cluster: **"Frontal-graphomotor cluster — Attention, executive control, and motor output co-vary and are jointly depressed, while verbal reasoning is spared."** The system explicitly labels this as the classic ADHD-Inattentive + dysgraphia signature, with convergence across 4/4 indicators and spared strengths (Verbal 102, Reasoning 96). This is a direct operationalization of the domain-profile-to-diagnosis mapping described below.

The Amber theme presents this as a "dark, single-purpose room for reading the whole profile at a glance" — a design philosophy emphasizing domain-level pattern recognition over score-by-score review. The [[summaries/redesign_20260623110817]] companion document describes additional visual design details.

### Live Synthesis Panel

The Synthesis Panel in the Report Workspace narrates domain findings in clinical language:

> *"Two findings converge on the referral question. Attention/Executive (SS 76) and graphomotor output are both depressed, while verbal reasoning is intact — a profile consistent with ADHD-Inattentive with co-occurring dysgraphia rather than a global delay."*

This live synthesis is produced by the [[concepts/clinical-ai-copilot]] (Luria) reasoning over the domain scores, and it explicitly uses the dissociation between depressed and spared domains as the primary interpretive logic — a direct application of the differential diagnostic principles described in the section below.

### Console & Synthesis: Domain-Level Differential Diagnosis

The Console module (`⌘4`) exposes domain-based reasoning most explicitly. In the redesign example, the clinician asks: *"Is the inattention primary ADHD, or secondary to the dysgraphia and academic frustration?"* Luria responds by invoking three domain-level lines of evidence:

1. **Cross-setting convergence** — Conners-4 inattention elevated on both parent and teacher forms (T=72/70); secondary inattention typically localizes to a single demanding setting
2. **Domain dissociation** — Attention/Executive (SS 76) depressed independently of graphomotor speed
3. **Developmental history** — Onset predates formal writing demands per caregiver interview

Each line of evidence is cited to a specific source (Conners-4, NEPSY-II, Clinical Interview). The provisional impression — **ADHD-Inattentive (primary), with co-occurring dysgraphia** — follows directly from the domain profile, and the response recommends CPT-3 and Beery VMI to complete domain coverage before finalizing. This [[concepts/clinical-ai-reasoning]] workflow operationalizes [[concepts/neuropsychological-synthesis]] at the domain level.

## Domain Coverage in Practice: A Single-Subtest Example

The [[summaries/NP-20240415-001_report]] illustrates an important edge case in domain-organized reporting: when only a single subtest is administered, the same result is applied across all domain sections of the report template. In that case, the WAIS-IV Digit Span (Raw Score: 16, Scaled Score: 9, 37th percentile, classified as Average) was entered under all six domain headings — Intelligence/General Cognitive Ability, Memory & Learning, Attention & Processing Speed, Executive Function, Language, and Visuospatial Ability — because the single instrument was the only available data source.

This scenario exposes a key limitation of domain-structured templates: when assessment scope is narrow, the domain scaffold can give a misleading impression of breadth. Best practice requires explicit disclosure of the limited evaluation scope and a caveat that most domains were not independently assessed. The Digit Span, as a measure of [[concepts/working-memory]], most naturally belongs to the Attention & Working Memory and Memory & Learning domains; its appearance in Language and Visuospatial sections in this report reflects template constraint rather than clinical inference.

The case also highlights the relationship between Digit Span performance and the Working Memory Index in the WAIS-IV: a scaled score of 9 (37th percentile, Average range) can nonetheless be flagged as clinically significant when contextual factors suggest a decline from estimated premorbid ability — a pattern consistent with [[concepts/mild-cognitive-impairment]], amnestic type.

## Real-World Domain Profiles: Clinical Intake Example

The [[summaries/multi_patient_transcript]] provides a naturalistic illustration of how cognitive domains present in a clinical intake setting, before formal testing. The patient — a man in his mid-30s with a longstanding history of speech and language disorder — shows a profile that cuts across several domains:

### Language
The most prominent and longstanding impairment. The patient was non-verbal until approximately age 4, never babbled, and acquired speech only through years of intensive therapy. Current presentation includes dysfluent output, word-finding difficulties, and flat/monotone prosody — consistent with a persistent motor speech disorder (speech dyspraxia).

### Executive Function
The intake highlights several features consistent with [[concepts/executive-function-deficits]]: impaired judgment (driving while exhausted), poor planning and foresight, cognitive inflexibility (difficulty adapting to changed workplace procedures), and impaired inhibition (compelled to address perceived disorder in colleagues' work). The father's analogy is apt: the patient knows the correct strategy but cannot execute it under real-world conditions — a dissociation between knowing and doing characteristic of executive dysfunction.

### Attention and Processing Speed
A consistent need for significantly greater intentional effort to perform routine cognitive tasks. This effort depletion accumulates across the day, leaving less cognitive reserve for safety-critical tasks such as driving. This domain is especially important clinically because attention weaknesses may be longstanding and developmental rather than newly acquired. As synthesized in [[summaries/attention-problems]], neuropsychological evaluation often becomes the point at which chronic but previously untreated attentional difficulties are first formally documented.

### Social Cognition
Mild impairment in reading subtle nonverbal cues, difficulty tracking complex social motives, preference for procedural over emotionally complex narratives, compulsive apologizing, and flat affect — consistent with autism spectrum features evaluated via a semi-structured rating instrument.

### Memory and Learning
Not formally tested in this session, but relevant history includes use of a smart recording pen throughout university and strong preference for visual learning modality.

### Motor / Sensorimotor
Fine motor difficulties documented from early childhood (pencil grip, scissors, buttons). Paradoxically, gross and proprioceptive motor control are excellent — suggesting domain-specific fine vs. gross motor dissociation.

### Adaptive Functioning
Currently semi-dependent on parents financially. Capable of structured routine tasks but struggling with novel, multi-step, or executive-heavy demands — characteristic of neurodevelopmental conditions that benefit substantially from environmental scaffolding.

## Supported Data Formats and Domain Columns

The Luria toolkit ingests neuropsychological data in CSV, Excel, JSON, Parquet, and SQLite formats. Regardless of source format, the expected tabular structure ties each score row explicitly to a cognitive domain:

```csv
subject_id,test_name,domain,raw_score,scaled_score,t_score,percentile
001,WAIS-IV_VCI,language,45,12,55,73
001,WAIS-IV_PRI,visuospatial,38,10,50,50
001,CVLT-II,memory,52,11,53,62
```

The `domain` column is the primary classification key. Downstream analysis — descriptive statistics, impairment classification, visualization, and report generation — all pivot on this field. The toolkit also supports [[concepts/long-format-clinical-data]] conventions, where each row represents a single test observation rather than a wide-format subject profile.

## Role in Structured Data Extraction

In the neuropsychological data extraction pipeline described in [[summaries/AGENTS_luria]], the `cognitive_domains_affected` field captures which domains show clinically significant findings for a given report. This field accepts a comma-separated list drawn from the canonical domains above.

This structured tagging enables:
- **Cross-report comparison** (e.g., tracking domain-specific decline over time)
- **Differential diagnosis support** (e.g., distinguishing amnestic MCI from dysexecutive presentations)
- **Research aggregation** across multiple assessments stored in tabular form

For attention findings specifically, structured extraction should ideally preserve whether the difficulty appears longstanding or acquired. The distinction summarized in [[summaries/attention-problems]] is clinically central and aligns with the broader framework of [[concepts/developmental-vs-acquired-cognitive-symptoms]].

## Role in Narrative Report Generation

Domains are not only classification tags — they are the primary organizational unit for written clinical narratives. The [[summaries/neuropsych-narrative-writer]] agent generates one Quarto include file per domain present in the extracted CSV data. For each domain, the narrative covers:

1. **Performance summary** — what was tested and the qualitative range of performance
2. **Pattern interpretation** — relative strengths and weaknesses, intra-test scatter, and score-type discrepancies
3. **Functional implication** — a hedged sentence linking domain performance to everyday, academic, or occupational impact

The [[summaries/neurocog.prompt]] configuration formalizes the prose register for cognitive domain summaries: third-person, past-tense voice; mixed technical and lay language leaning professional; complete and sophisticated paragraph-form output with pronoun-sensitive patient references. This approach operationalizes [[concepts/clinical-communication-register]] for each domain section.

For multi-rater domains such as ADHD and Emotion/Behavior, separate files are produced per rater (self, parent, teacher) — but only for raters with actual data present. Missing raters are skipped entirely; data is never fabricated.

This domain-by-domain prose structure integrates with the [[concepts/modular-report-architecture]] of the cingulate Quarto/Typst pipeline, where the R6 layer handles score tables and plots while the narrative agent handles prose exclusively. The Luria toolkit's report generation layer supports output templates scoped by population (Adult, Pediatric, Forensic, Research), each of which selects appropriate domain coverage and clinical register.

## Luria Toolkit: Domain-Aware API and CLI

As documented in [[summaries/README]] and [[summaries/README_luria]], Luria provides several entry points for domain-aware analysis:

- **CLI**: `luria analyze --domain memory` or `luria report --domains memory,attention,executive`
- **Python API**: `import luria; luria.compute_descriptive_stats(domains=['memory', 'attention'])`
- **R Integration**: The `cingulate` R package bridges domain-structured data from Luria's Python backend into R-based visualizations and report templates
- **Batch Processing**: Multiple datasets can be processed simultaneously, each returning per-domain summary statistics

## Analysis Capabilities by Domain

Once data is organized by domain, the Luria toolkit supports a layered analysis stack:

- **Descriptive statistics**: Mean, SD, range, and percentile distributions per domain
- **Impairment classification**: Cutoff-based or normative-comparison-based flagging
- **Profile analysis**: Intra-individual domain comparisons highlighting relative strengths and weaknesses
- **Statistical tests**: T-tests, ANOVA, correlation, and regression across domains
- **Normative comparisons**: Benchmarking scores against standardization samples
- **Visualization**: Domain heatmaps, score profile plots, constellation maps, and longitudinal trajectory charts

The R-Python integration layer — Luria's Python backend paired with the `cingulate` R package — enables these analyses to be executed either programmatically or through the CLI, with results fed directly into report templates.

## Relationship to Test Scores

Each cognitive domain is operationalized through one or more standardized tests or subtests. The mapping is many-to-many:
- A single test (e.g., Trail Making Test B) may index both **Processing Speed** and **Executive Function**
- A single domain (e.g., Memory) may be assessed by multiple instruments (e.g., CVLT-3, WMS-IV, RBANS Memory Index)
- A single subtest (e.g., WAIS-IV Digit Span) may be reported across multiple domain sections when no other data is available

For details on how individual scores are captured and structured, see [[concepts/neuropsychological-test-scores]].

## Clinical Significance

### Domain Profiles and Diagnosis
The pattern of preserved versus impaired domains is diagnostically meaningful:
- **Amnestic MCI**: Isolated or predominant Memory impairment — see [[concepts/mild-cognitive-impairment]]
- **Alzheimer's Disease**: Memory + Language + Visuospatial decline
- **Frontal/Executive Syndrome**: Executive Function + Attention + Processing Speed
- **Posterior Cortical Atrophy**: Visuospatial + Language with relative memory sparing
- **Autism Spectrum Disorder (High-Functioning)**: Social Cognition impairment alongside relative preservation of other domains; may co-occur with language, executive, and adaptive functioning difficulties — as illustrated in [[summaries/multi_patient_transcript]] and contextualized by [[concepts/autism-spectrum-disorder-clinical-features]]
- **ADHD-Inattentive with co-occurring dysgraphia**: Attention, executive control, and graphomotor output co-depressed; verbal reasoning and general cognitive ability intact — the frontal-graphomotor cluster pattern identified by the Luria Cognitive Map in [[summaries/redesign_20260623110910]]
- **Neurodevelopmental Language Disorder**: Primary Language domain impairment with cascading effects on attention, processing speed, and adaptive functioning

Attention-domain findings deserve special caution. Weakness in sustained, selective, divided, or working-memory-linked attention does not by itself establish whether the problem is developmental or acquired. As emphasized in [[summaries/attention-problems]], some neuropsychological reports document attentional weaknesses as longstanding across the lifespan but only first treated after a later clinical event. This makes developmental history essential to domain interpretation.

### Premorbid vs. Acquired Attention Difficulty Within Domain Analysis
Attention is one of the clearest examples of why domain scoring alone is insufficient. Similar test performance can arise from very different etiologies:
- **Longstanding attentional weakness** associated with ADHD, dyslexia, or other neurodevelopmental conditions
- **Acquired attentional decline** following traumatic brain injury, stroke, or other CNS conditions
- **Secondary attentional inefficiency** due to psychiatric symptoms, fatigue, pain, or academic frustration

Accordingly, domain-based neuropsychological interpretation should integrate test data with developmental history, collateral report, timing of onset, and functional trajectory. This logic aligns directly with [[concepts/premorbid-vs-acquired-attention-difficulties]] and the broader framework of [[concepts/developmental-vs-acquired-cognitive-symptoms]].

### The Knowing–Doing Dissociation
A clinically important pattern highlighted in the [[summaries/multi_patient_transcript]] intake is the dissociation between domain-level knowledge and domain-level action. The patient can articulate correct strategies (tennis tactics, driving safety, workplace procedures) but cannot reliably implement them under real-world conditions. This gap between knowing and doing is a hallmark of [[concepts/executive-function-deficits]] and is distinct from deficits in declarative memory or language comprehension.

### Working Memory as a Cross-Domain Construct
The Working Memory domain occupies an especially prominent position in assessment because it overlaps multiple domains: it contributes to Attention (auditory sequential processing), Memory & Learning (short-term retention), and Executive Function (active manipulation of information). Instruments such as the WAIS-IV Digit Span serve as markers of [[concepts/working-memory]] and contribute to the Working Memory Index composite. A score in the average range (e.g., scaled score of 9, 37th percentile) may still carry clinical weight when interpreted against estimated premorbid ability or in the context of subjective cognitive complaints.

### Discrepancy Analysis
Large discrepancies between domain scores (e.g., high Verbal Comprehension vs. low Processing Speed) are flagged as `notable_findings` in the extraction schema — a clinically important signal for conditions such as ADHD, traumatic brain injury, or learning disabilities. In neurodevelopmental presentations, cross-domain scatter (e.g., strong visuospatial and mathematical reasoning alongside severely impaired language production) can mask overall ability when composite scores are used without decomposition.

### Age-Group Register
Clinical narratives about domains are tailored to the patient population. The `age_group` column in extracted data drives register selection: pediatric language for child reports, adult language for adult reports, and a forensic register for forensic evaluations. See [[concepts/forensic-neuropsychological-evaluation]] for the forensic context.

### Longitudinal Tracking
When reports are collected over time, domain-level scores can be compared across timepoints to track cognitive trajectories — a key use case supported by the optional `timepoint` field in the [[summaries/AGENTS_luria]] schema and by the Luria toolkit's longitudinal visualization features. In neurodevelopmental cases, longitudinal tracking is especially important: the [[summaries/multi_patient_transcript]] case illustrates substantial growth over decades, particularly in language and social domains, attributable in part to intensive early intervention and sustained family scaffolding.

## Integration with NLP Pipelines

Automated extraction of domain classifications from clinical text is a key challenge in [[concepts/clinical-nlp-pipelines]]. Domain labels must often be inferred from test names and qualitative descriptions rather than stated explicitly, requiring contextual reasoning about which constructs each instrument measures. Clinical intake transcripts such as [[summaries/multi_patient_transcript]] present additional challenges: domain-relevant information is embedded in natural conversation, across multiple informants, without formal test scores — requiring inference from behavioral descriptions to likely domain impairment.

Attention findings create an additional NLP challenge: systems must not only identify that the attention domain is affected, but also preserve whether the narrative frames the problem as longstanding, newly emerged, or worsened after a triggering event. The recurring pattern described in [[summaries/attention-problems]] shows why this distinction matters for downstream synthesis and recommendation generation.

The document ingestion pipeline described in the Luria redesign ([[summaries/redesign_20260623110910]]) uses RAG (24 embedded chunks) to extract grounded facts from clinical records, linking each extracted fact to its source page. These facts — including domain-relevant findings such as "Expressive speech delay treated age 3-4" — pre-fill the report background and inform the AI synthesis layer, embodying the [[concepts/retrieval-augmented-generation]] approach to [[concepts/clinical-data-management]].

## Related Pages
- [[summaries/AGENTS_luria]] — Source extraction schema defining the domain taxonomy
- [[summaries/README_luria]] — Luria toolkit overview including domain-aware CLI and API
- [[summaries/README]] — Top-level Luria documentation hub linking all domain-related guides and tutorials
- [[summaries/neuropsych-data-extractor]] — Stage 2 pipeline that produces the CSV consumed by the narrative agent
- [[summaries/neuropsych-narrative-writer]] — Stage 3 agent that writes per-domain narrative includes
- [[summaries/NP-20240415-001_report]] — Example report illustrating single-subtest domain coverage
- [[summaries/multi_patient_transcript]] — Clinical intake illustrating naturalistic multi-domain presentation
- [[summaries/neurocog.prompt]] — Prompt configuration for cognitive domain narrative generation
- [[summaries/neurobehav.prompt]] — Companion prompt for behavioral and emotional domain interpretation
- [[summaries/redesign_20260623110910]] — Luria.app redesign illustrating domain visualization in the Cognitive Map and Synthesis panel
- [[summaries/attention-problems]] — Distinguishing longstanding from acquired attention-domain difficulties
- [[concepts/neuropsychological-test-scores]] — How individual test results map to domains
- [[concepts/neuropsychological-reporting]] — Broader context of clinical reporting
- [[concepts/luria-neuropsych-pipeline]] — The multi-stage pipeline in which domain taxonomy is used
- [[concepts/modular-report-architecture]] — Quarto/Typst structure that organizes output by domain
- [[concepts/clinical-nlp-pipelines]] — Automated extraction of domain data from text
- [[concepts/phi-data-handling]] — Privacy considerations when processing domain-level clinical data
- [[concepts/narrative-report-generation]] — How domains translate into structured prose output
- [[concepts/long-format-clinical-data]] — Row-per-observation data structure used in Luria ingestion
- [[concepts/working-memory]] — Cross-domain construct measured by instruments such as WAIS-IV Digit Span
- [[concepts/mild-cognitive-impairment]] — Diagnostic category frequently identified through domain profile analysis
- [[concepts/forensic-neuropsychological-evaluation]] — Forensic register for domain-level clinical narratives
- [[concepts/executive-function-deficits]] — Knowing–doing dissociation and real-world behavioral breakdown
- [[concepts/autism-spectrum-disorder-clinical-features]] — Social cognition and restricted interest domain profiles
- [[concepts/clinical-communication-register]] — Register calibration for professional and lay audiences
- [[concepts/dual-audience-design]] — Balancing technical and accessible language in domain narratives
- [[concepts/neuropsychological-prompt-configuration]] — Model parameter settings for domain summary generation
- [[concepts/clinical-ai-copilot]] — AI reasoning layer that produces live domain-level synthesis
- [[concepts/clinical-ai-reasoning]] — Evidence-based differential diagnosis from domain profiles
- [[concepts/neuropsychological-synthesis]] — Integration of domain findings into clinical formulation
- [[concepts/retrieval-augmented-generation]] — Document ingestion and grounded fact extraction in the clinical pipeline
- [[concepts/local-llm-inference]] — PHI-safe inference supporting domain-level narrative generation
- [[concepts/dysgraphia]] — Motor/graphomotor domain impairment co-occurring with ADHD-Inattentive
- [[concepts/premorbid-vs-acquired-attention-difficulties]] — Attention-domain interpretation across developmental vs acquired causes
- [[concepts/developmental-vs-acquired-cognitive-symptoms]] — Broader framework for distinguishing lifelong from newly emergent weakness

See also: [[summaries/SKILL]] | [[summaries/template-system]] | [[summaries/brainworkup-brand-voice-guide]] | [[summaries/report_body]]

See also: [[summaries/CLAUDE]] | [[summaries/LLM_AGENT_MAP]] | [[summaries/LLM_INTEGRATION]]

See also: [[summaries/nt_interpretation]]

See also: [[summaries/sirf_synthesis]]

See also: [[summaries/CARS2-Manual_extracted]]

See also: [[summaries/bwu.neuro.reports.recs.build-math-skills]]

See also: [[summaries/bwu.neuro.reports.recs.building-verbal-skills]]

See also: [[summaries/redesign_20260623110817]]

See also: [[summaries/clinical-assessment]]