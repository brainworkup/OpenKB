---
sources: [summaries/MIGRATION_GUIDE.md]
brief: Training a dedicated voice profile per report section captures nuances better than one global style.
---

# Voice Profiles: Section-Specific vs Global
This concept addresses whether to train a single writing voice for every generated draft or to specialize by report section — and which approach yields the highest quality.

## The Problem with One Global Profile
A single "soul profile" (trained on 25 examples) attempts to capture multiple distinct voices simultaneously: the Clinical Narrative, NSE, TESTING, and RECOMMENDATIONS sections all have different purposes, target audiences, and conventions. Smearing them into one averaged voice blurs each section's strengths.

## The Section-Specific Approach
Instead of a global profile, train four dedicated profiles — one for each canonical section of the neuropsychological report. Each is trained on 25 examples from its specific context:
- **NSE**: therapeutic goals and outcomes
- **TESTING**: data presentation and interpretation
- **SUMMARY**: executive overview
- **RECOMMENDATIONS**: actionable, patient-facing advice

**The Workflow:**
1. Train all four profiles (via `train_section_profiles.sh` or manual commands).\
2. Use the matching profile for each section during generation: e.g., `--profile-path ./soul_db/report_soul_profile.recommendations.json --section RECOMMENDATIONS`.

## Why It Works Better\Each recommendation draft is generated against previous recommendations; each NSE summary against prior NSE summaries. This ensures the voice in a new section matches the historical register of that specific part of the report, rather than an average across unrelated sections.\ The system falls back to cosine similarity if FAISS isn't present and handles docling/PyMuPDF automatically, so this architectural choice is decoupled from tooling dependencies.\n
See [[concepts/style-profiles]] for broader voice strategy and [[summaries/MIGRATION_GUIDE]] for the operational details on training each profile.