---
doc_type: short
full_text: sources/QUICK_REFERENCE.md
---

# Quick Reference: Age Override System

A concise operational guide for managing **age overrides** in the report ingestion pipeline. Age corrections are permanent, stored in `data/age_overrides.json`, and automatically applied during ingestion.

## Key Files

| File | Purpose |
|------|---------|
| `data/age_overrides.json` | Edit to add/change age overrides |
| `src/report_parser.py` | Auto-loads overrides during parse |
| `AGE_OVERRIDE_GUIDE.md` | Full documentation |
| `PERMANENT_SOLUTION_SUMMARY.txt` | Solution overview |

## Core Workflow

1. **Add override**: Edit `data/age_overrides.json` with `{ "overrides": { "report.pdf": 28 } }`
2. **Run ingestion**: `python -m src.ingest_recommendations`
3. **Verify**: Query `data/recommendations_kb.json` to confirm `age_group` field

## [[concepts/age-group-classification]]

Age groups are assigned based on numeric thresholds:

| Age Range | Group |
|-----------|-------|
| 0–12 | pediatric |
| 13–17 | adolescent |
| 18–64 | adult |
| 65+ | geriatric |

Overrides only apply when the parser **cannot find an age in the PDF**. If an age is present in the source document, it takes precedence.

## Current Dataset Status

- ✅ 11 reports have overrides configured
- ✅ 2 reports remain `unknown` (duplicates/templates with no patient data)
- ✅ **98.4%** of the dataset has known age groups

## Troubleshooting

- Filenames are **case-sensitive** — must match exactly
- Validate JSON with: `python -m json.tool data/age_overrides.json`
- If override isn't applied, confirm file path and re-run ingestion
- Reports still showing `unknown` after override: check if age exists in PDF (PDF value takes precedence)

## Related

- [[concepts/age-group-classification]] — age threshold logic
- [[concepts/report-ingestion-pipeline]] — overall ingestion workflow
- [[concepts/report-parser-quality]] — parser reliability and override strategies
- [[concepts/clinical-data-management]] — broader data management context

## Related Concepts
- [[concepts/neuropsych-report-parsing]]
- [[concepts/phi-data-handling]]
