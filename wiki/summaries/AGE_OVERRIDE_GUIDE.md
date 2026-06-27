---
doc_type: short
full_text: sources/AGE_OVERRIDE_GUIDE.md
---

# AGE_OVERRIDE_GUIDE — Summary

## Overview

This guide documents the **Age Override System** — a mechanism for permanently correcting `age_group: "unknown"` entries that arise when the report parser cannot automatically extract patient age from neuropsychological PDF reports during ingestion.

## Problem Context

When running `python -m src.ingest_recommendations`, some PDFs lack extractable age data. Without age, reports are classified as `age_group: "unknown"`, which limits downstream filtering and analysis in the knowledge base. This relates directly to [[concepts/age-group-classification]] and the broader [[concepts/neuropsychological-assessment-pipeline]].

## How It Works

1. **Override file**: `data/age_overrides.json` stores filename-to-age mappings.
2. **Parser integration**: `src/report_parser.py` checks the override file as a fallback when automatic extraction fails.
3. **Age group classification** is automatic per [[concepts/age-group-classification]]:
   - `< 13` → `pediatric`
   - `13–17` → `adolescent`
   - `18–64` → `adult`
   - `65+` → `geriatric`

## Extraction Priority

The parser tries age extraction in this order:
1. Header line (e.g., `PATIENT NAME: John Doe, Age 25`)
2. Body text patterns (e.g., "is a 25-year-old")
3. **Fallback**: `data/age_overrides.json`

Note: If the PDF itself contains an age, that takes precedence over any override. This fallback pattern is an instance of the [[concepts/fallback-strategy]] used throughout the pipeline.

## Adding Overrides

Two methods:
- **Direct JSON edit** of `data/age_overrides.json`
- **Python script** to load, update, and save the JSON file

## Current Overrides (as of 2026-02-11)

| Report | Age | Age Group |
|--------|-----|-----------|
| aria_dewey_neuro_report_2023_04_07.pdf | 23 | adult |
| malan_annette_neuropsychological_report_2023_03_02.pdf | 19 | adult |
| neuropsychological_report_writing___r2_digital_library_joe_bruin.pdf | 13 | adolescent |
| npsych_report_bachsihan_kol_2021_09_16.pdf | 8 | pediatric |
| npsych_report_brown_nicholas_2021_10_28.pdf | 26 | adult |
| npsych_report_kim_albert_2021_05_13.pdf | 29 | adult |
| npsych_report_valdez_douglas_2021_05_04.pdf | 31 | adult |
| npsychreport_whitakerstella_2022_01_13.pdf | 8 | pediatric |
| sourina_neuropsychological_report_2023_03_09.pdf | 42 | adult |
| young_summer_neuropsychological_report_2023_02_03.pdf | 24 | adult |
| zhang_aj_neuropsychological_report_2023_01_19.pdf | 19 | adult |

Two reports remain `unknown` by design (duplicates/templates without patient info): `autumn_w_summer_data.pdf` and `donders__neuropsychological_report_writing_examplereport_2.pdf`.

## Key Files

- `data/age_overrides.json` — override configuration
- `src/report_parser.py` — modified to include `load_age_overrides()` and updated `extract_report_metadata()`
- `data/recommendations_kb.json` — output knowledge base checked for unknown age groups

See also [[summaries/neuropsych-pdf-parser]] for related parser documentation and [[concepts/pdf-data-extraction]] for the broader extraction context.

## Ingestion Workflow

1. Add PDF to the bucket directory
2. Run `python -m src.ingest_recommendations`
3. Check for `unknown` age groups
4. If needed, add age override and re-run

This workflow integrates with the [[concepts/neuropsychological-assessment-pipeline]] and relies on [[concepts/clinical-data-management]] practices to ensure patient records are correctly classified.

## Troubleshooting

- Filenames in the override file are **case-sensitive**
- JSON syntax can be validated with `python -m json.tool data/age_overrides.json`
- File must be at `data/age_overrides.json` relative to project root
- Re-ingestion is required for changes to take effect

## Potential Enhancements

- CLI command for adding overrides
- Startup validation of override file
- Logging when overrides are applied (see [[concepts/decision-logging]])
- Fuzzy filename matching support

## Related Concepts
- [[concepts/pdf-score-extraction]]
- [[concepts/luria-neuropsych-pipeline]]
- [[concepts/clinical-nlp-pipelines]]
- [[concepts/docling-pdf-parsing]]
- [[concepts/edit-protection-pattern]]

- [[concepts/age-group-classification]]
- [[concepts/fallback-strategy]]
- [[concepts/neuropsychological-assessment-pipeline]]
- [[concepts/pdf-data-extraction]]
- [[concepts/clinical-data-management]]
- [[concepts/phi-data-handling]]