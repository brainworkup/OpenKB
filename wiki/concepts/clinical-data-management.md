---
sources: [summaries/cerner-autotext.md, summaries/SESSION_SUMMARY.md, summaries/QUICK_REFERENCE.md, summaries/AGE_OVERRIDE_GUIDE.md, summaries/README.md, summaries/responses_to_claude.md, summaries/processed_files.md]
brief: Managing clinical files, structured content, and data flows across care workflows.
---

# Clinical Data Management

Clinical data management encompasses the strategies, workflows, and technical practices used to store, organize, retrieve, govern, and reuse clinical information across documentation and assessment systems. In practice, this includes not only neuropsychological assessment PDFs, score exports, and associated reports, but also structured EHR content such as Cerner Autotext tokens that dynamically populate patient-specific fields in clinical notes. Across academic medical centers and private practices, the core challenge is coordinating heterogeneous data sources while preserving accuracy, provenance, privacy, and usability.

## The Problem: Data Sprawl

Clinical data, particularly assessment output files like PAI reports (see [[concepts/pai-assessment]]), tends to accumulate across multiple disjointed locations over time. A concrete example from neuropsychological practice illustrates this well: a single `pai.pdf` file may exist in 208 locations spanning:

- Local archived directories organized by year and patient name
- Active report folders on network drives
- Cloud storage mounts (OneDrive, Dropbox, Google Drive) — sometimes with both a CloudStorage symlink path and a Group Containers path representing the same underlying data
- Knowledge base and reference directories
- Downloads folders and even the system Trash

This kind of sprawl emerges organically as clinicians move files between workflows, create versioned subdirectories for longitudinal cases, and sync data across platforms.

A parallel form of sprawl occurs inside EHR documentation itself. In Cerner, patient facts may be inserted through Autotext tokens rather than manually typed text, meaning the effective data source for a note may be distributed across templated token libraries, structured chart fields, and rendered narrative output. This adds a second management layer: not just where files live, but how structured patient data is referenced and resolved during note generation.

## Consolidation Strategies

A common remediation approach is scripted bulk consolidation — using shell scripts to recursively search the filesystem and copy all instances of a target file type into a single canonical directory, renaming files sequentially. This enables downstream processing such as [[concepts/pdf-score-extraction]], [[concepts/ocr-pipeline]], and batch scoring pipelines.

At the documentation layer, consolidation also means reducing redundant manual entry by using standardized templates and EHR-linked field insertion. Cerner Autotext exemplifies this approach: bracketed tokens can populate names, pronouns, demographics, dates, care-team fields, medication lists, allergies, social history, and problem-list content directly from structured sources. From a data-management perspective, this improves consistency and lowers transcription burden, while also making note content dependent on the integrity of underlying structured fields.

**Key considerations when consolidating:**

- **Deduplication**: Cloud sync tools often mount the same data at multiple paths (e.g., `CloudStorage/OneDrive-...` and `Group Containers/UBF8T346G9.OneDriveStandaloneSuite/...`), producing apparent duplicates that are actually mirrors. True deduplication requires hash comparison, not just filename matching.
- **Versioning awareness**: Patients seen longitudinally may have numbered subdirectories (e.g., `Tianyu/Tianyu2/Tianyu3` or `Lauren/Lauren7/Lauren5/Lauren2`) representing successive assessment sessions. Consolidation scripts should preserve or log provenance so the chronological sequence can be reconstructed.
- **IME vs. clinical distinction**: Independent Medical Examination (IME) cases and standard clinical evaluations may be stored in separate folder hierarchies and should be tagged or separated during consolidation.
- **Trash recovery**: Files deleted to the system Trash may still contain valid clinical data and should be considered during comprehensive consolidation sweeps.
- **Template-field reliability**: In EHR-based workflows, tokenized fields may not always resolve correctly. The Cerner Autotext example notes that an age token may be non-functional, with date of birth preferred instead. Data management therefore includes maintaining practical knowledge of unreliable fields and approved substitutions.
- **Structured-to-narrative provenance**: When note text is auto-populated from EHR fields, the source of each populated value should be understood so discrepancies between rendered notes and chart data can be traced.

## Storage Architecture Patterns

Neuropsychological practices commonly use a tiered storage model:

| Tier | Example Paths | Role |
|------|--------------|------|
| Local archive | `~/cortex/archived_neuropsychological_reports/` | Long-term cold storage by year |
| Active reports | `~/reports/` | Current workflow files |
| Institutional cloud | OneDrive (Keck Medicine of USC) | Compliance, sharing, backup |
| Personal cloud | Google Drive, Dropbox | Personal backup and reference |
| Knowledge base | `~/neuropsychology/knowledgeBase/measures/` | Canonical reference copies |
| Shared clinical resources | `clinical_resources/books/` | Reference books and textbooks shared across all patient RAG systems |
| EHR structured content | Cerner chart fields and Autotext libraries | Canonical source for reusable demographics, care-team, and note-insertable patient data |

This broader view of storage architecture highlights that some clinical data is managed as files, while other data is managed as structured fields resolved into narrative text at documentation time.

## Shared Clinical Reference Materials

Beyond per-patient files, clinical data management also encompasses shared reference resources used across all patient workflows. The `clinical_resources/books/` directory exemplifies this pattern: it holds clinical reference books and textbooks (PDF, DOCX, or text files) that are automatically ingested whenever any patient's RAG system is rebuilt. This centralized approach ensures consistent access to authoritative reference knowledge across multiple patient-specific retrieval systems, and it supports the [[concepts/retrieval-augmented-generation]] architecture used for clinical decision support.

Best practices for this shared layer include:
- Using descriptive filenames for all reference materials
- Storing files in widely supported formats (PDF, DOCX, plain text)
- Reviewing and updating the directory regularly to reflect current clinical standards

A similar standardization principle applies to EHR documentation assets such as note templates and Autotext token reference lists: centrally maintained reusable components improve consistency across clinicians and reports, especially for recurring sections like medications, allergies, social history, and problem lists.

## Privacy and Compliance Implications

Because clinical files contain protected health information (PHI), data consolidation scripts must be operated with care. Running such scripts with elevated privileges (`sudo`) on machines that also sync to personal cloud accounts creates risks of PHI co-mingling with non-institutional storage. See [[concepts/phi-data-handling]] and [[concepts/clinical-data-privacy]] for related guidance.

The same privacy concerns apply to structured EHR-derived documentation. Autotext tokens can automatically insert names, MRNs, dates of birth, medications, and other sensitive patient data into notes, which means exported, copied, or repurposed documentation may contain PHI even when the clinician did not manually type those details. Managing clinical data therefore requires awareness of both file-level PHI exposure and template-driven PHI propagation.

Redaction and de-identification workflows (see [[concepts/pii-redaction-pipelines]]) should be applied before any consolidated files or rendered notes are used in research, ML training, or knowledge base construction. Shared reference materials that do not contain PHI (e.g., textbooks and clinical manuals) carry lower risk but should still be managed under appropriate access controls.

## Relationship to Downstream Pipelines

Consolidated clinical PDFs serve as the input layer for several downstream processes:

- [[concepts/pdf-score-extraction]] — parsing numerical scores from assessment output PDFs
- [[concepts/ocr-pipeline]] — converting scanned or image-based PDFs to machine-readable text
- [[concepts/neuropsychological-assessment-pipeline]] — end-to-end workflows from raw data to structured report
- [[concepts/pai-knowledge-base]] — building a queryable knowledge base from PAI assessment data
- [[concepts/clinical-pdf-assessment]] — systematic review and interpretation of PDF-format clinical instruments
- [[concepts/retrieval-augmented-generation]] — shared reference materials feeding patient-specific RAG systems

Structured EHR content feeds a parallel downstream pathway. Autotext-populated note fields support standardized clinical narrative generation, improve documentation efficiency, and help connect chart data to rendered reports. In this sense, clinical data management underlies both file-centric assessment pipelines and narrative documentation workflows, including [[concepts/neuropsychological-reporting]] and [[concepts/narrative-report-generation]].

## Related Concepts

- [[concepts/pai-assessment]]
- [[concepts/neuropsychological-reporting]]
- [[concepts/narrative-report-generation]]
- [[concepts/phi-data-handling]]
- [[concepts/clinical-data-privacy]]
- [[concepts/pii-redaction-pipelines]]
- [[concepts/pdf-score-extraction]]
- [[concepts/ocr-pipeline]]
- [[concepts/parquet-as-knowledge-store]]
- [[concepts/retrieval-augmented-generation]]
- [[concepts/knowledge-base-architecture]]

## Related Documents

- [[summaries/cerner-autotext]]
- [[summaries/processed_files]]
- [[summaries/README]]
- [[summaries/AGE_OVERRIDE_GUIDE]]
- [[summaries/QUICK_REFERENCE]]
- [[summaries/SESSION_SUMMARY]]
- [[summaries/responses_to_claude]]