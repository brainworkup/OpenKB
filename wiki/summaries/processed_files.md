---
doc_type: short
full_text: sources/processed_files.md
---

# processed_files: PAI Report Consolidation Script Output

## Overview

This document is the terminal output of a shell script (`find_and_copy_pai.sh`) run with `sudo` privileges in a conda environment. The script performed a system-wide search for files named `pai.pdf` and copied all found instances into a single consolidated directory at `/Users/joey/pai/reports/`, renaming them sequentially (`pai_1.pdf` through `pai_195.pdf`, plus additional files up to `pai_208.pdf`).

**Total files processed: 208**

## Purpose

The script appears designed to aggregate [[concepts/pai-assessment]] report PDFs that had accumulated across many disorganized locations into one central repository for analysis or review. This is a foundational step in building the [[concepts/pai-knowledge-base]].

## Source Locations

The PAI PDF files were found scattered across a wide range of directories, including:

### Local Archived Neuropsychological Reports
- `/Users/joey/cortex/archived_neuropsychological_reports/` — years 2021, 2022, 2023, 2024
  - Patients: Limas-Nathan, StPierre-Lars, Barreto-Alejandra, Patel-Krishan, Jemiel, Bridget, Reiner, Michelle, Amer-Alyssa, Bugacov-Helena
- IME (Independent Medical Exam) cases: Daniel (2024), Nestor (2023)
- USC clinical cases: AJ, Sourina, Alexandra, Aria, Annette, Summer, Anna

### OneDrive — Keck Medicine of USC
- `/bwu_neuropsych_reports/` (active reports)
- `/bwu_neuropsych_reports/x.archived.reports/` (archived)
- `/neuropsychological.reports/`
- `/backup/2023/archived_neuropsych_reports/`

Patients appearing across these: Alex, Krishan, Max, Leslie, Lars, Adam, Tianyu, Alejandra, Alexander, Alexander2, Lauren, AJ, Sourina, Lilith, Dylan, Woody, Alexandra, Charles, Aria, Kira, Nathan, Annette, Summer, Anna, Tiffany, Ashish, Raven, Gorety, Benjamin, Corey, Elea, Christine, Nestor

### Group Containers / OneDrive Standalone
- Mirror of OneDrive — Keck Medicine of USC structure (pai_108 through pai_189)

### Other Locations
- `/Users/joey/.Trash/` — recovered from Trash (pai_190–192)
- `/Users/joey/Downloads/` (pai_193)
- `/Users/joey/pai/source/` and `/Users/joey/pai/reports/` (pai_194–195)
- `/Users/joey/wm/data/pdf/` (pai_23)
- `/Users/joey/neuropsychology/knowledgeBase/measures/PAI/` (pai_22)
- `/Users/joey/usc/02_clinical_activities_patient_care/` faculty practice measures (pai_21)
- `/Users/joey/Library/CloudStorage/GoogleDrive-j.trampush@gmail.com/` knowledge base (pai_24)
- `/Users/joey/Library/CloudStorage/Dropbox/reports/` (pai_25)
- `/Users/joey/reports/` — active reports directory (pai_196–208)
  - Eric, Lilith, Dylan, Edward, Corey, Raven, Elea, Itamar, Tiffany, Gorety, Ashish

## Key Observations

- **Significant duplication**: Many patients appear multiple times across OneDrive CloudStorage and OneDrive Group Containers (which are likely synced mirrors), resulting in duplicate copies.
- **Versioning pattern**: Several patients have numbered subdirectories (e.g., `Tianyu/Tianyu2/Tianyu3`, `Lauren/Lauren7/Lauren5/Lauren2`) suggesting longitudinal reassessments or report revisions.
- **IME vs. clinical**: A distinction exists between standard neuropsychological evaluations and IME (Independent Medical Examination) cases.
- **Multi-platform storage**: Files span local disk, Dropbox, Google Drive, and two OneDrive mount points (CloudStorage symlink and Group Containers), illustrating the challenges of [[concepts/clinical-data-management]] across heterogeneous storage systems.
- **Trash recovery**: Three PAI files were recovered from the system Trash.
- **PHI exposure risk**: The widespread scatter of patient-named directories containing clinical PDFs across multiple cloud sync services raises concerns addressed by [[concepts/phi-data-handling]] and [[concepts/clinical-data-privacy]].

## Related Concepts

- [[concepts/pai-assessment]]
- [[concepts/pai-knowledge-base]]
- [[concepts/neuropsychological-assessment-pipeline]]
- [[concepts/clinical-data-management]]
- [[concepts/clinical-data-privacy]]
- [[concepts/phi-data-handling]]
- [[concepts/pdf-score-extraction]]
- [[concepts/clinical-pdf-assessment]]