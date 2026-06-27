---
doc_type: short
full_text: sources/PERMANENT_SOLUTION_SUMMARY.md
---

# Permanent Age Override Solution Summary

## Overview

This document describes the implementation of a **persistent age override configuration system** for a neuropsychological report ingestion pipeline. The core problem was that running `python -m src.ingest_recommendations` would overwrite cleaned data, causing manually corrected age group assignments to be lost on each re-ingestion.

## Problem Solved

Each re-ingestion of PDF-based neuropsychological reports would reset age metadata, losing corrections to [[concepts/age-group-classification]] that had been applied manually. The solution introduces a JSON-based override file that persists across re-ingestions.

## Files Created or Modified

| File | Action | Purpose |
|---|---|---|
| `data/age_overrides.json` | Created | Stores 11 report → age mappings |
| `src/report_parser.py` | Modified | Loads and applies overrides during parsing |
| `AGE_OVERRIDE_GUIDE.md` | Created | Documentation and usage instructions |

## Ingestion Flow

The modified [[concepts/report-ingestion-pipeline]] follows this logic:

1. Parse the PDF report
2. Try to extract age from the header (e.g., `Name, Age 25`)
3. Try to extract age from body text (e.g., `25-year-old`)
4. If age is still `None`, consult `data/age_overrides.json`
5. Classify extracted age into an age group

## Age Classification Rules

The system maps numeric ages to categorical [[concepts/age-group-classification]] groups:

- **Pediatric**: age < 13
- **Adolescent**: age 13–17
- **Adult**: age 18–64
- **Geriatric**: age 65+
- **Unknown**: age not determinable

## Current Override Mappings (11 Reports)

### Pediatric (age 8)
- `npsych_report_bachsihan_kol_2021_09_16.pdf`
- `npsychreport_whitakerstella_2022_01_13.pdf`

### Adolescent (age 13)
- `neuropsychological_report_writing___r2_digital_library_joe_bruin.pdf`

### Adult (ages 19–42)
- `aria_dewey_neuro_report_2023_04_07.pdf` → 23
- `malan_annette_neuropsychological_report_2023_03_02.pdf` → 19
- `npsych_report_brown_nicholas_2021_10_28.pdf` → 26
- `npsych_report_kim_albert_2021_05_13.pdf` → 29
- `npsych_report_valdez_douglas_2021_05_04.pdf` → 31
- `sourina_neuropsychological_report_2023_03_09.pdf` → 42
- `young_summer_neuropsychological_report_2023_02_03.pdf` → 24
- `zhang_aj_neuropsychological_report_2023_01_19.pdf` → 19

## Adding New Overrides

New overrides can be added by directly editing `data/age_overrides.json` or via a Python snippet. After editing, re-running the ingestion script applies the changes automatically.

## Key Benefits

- **Permanent**: Corrections survive re-ingestion cycles
- **Automatic**: No manual intervention needed after initial setup
- **Transparent**: All overrides centralized in one file
- **Non-invasive**: Original PDFs remain unchanged
- **Flexible**: Simple JSON format allows easy additions or removals

## Related Concepts
- [[concepts/edit-protection-pattern]]
- [[concepts/neuropsychological-assessment-pipeline]]
- [[concepts/yaml-configuration]]
- [[concepts/phi-data-handling]]

- [[concepts/age-group-classification]] — The taxonomy used to group patients by age
- [[concepts/report-ingestion-pipeline]] — The broader system for parsing and storing neuropsychological report data
- [[concepts/neuropsych-report-parsing]] — Techniques for extracting structured metadata from neuropsychological report PDFs
- [[concepts/recommendation-rag-pipeline]] — The RAG system that consumes the ingested report knowledge base
- [[concepts/clinical-data-management]] — Broader practices for maintaining clinical data integrity across processing cycles