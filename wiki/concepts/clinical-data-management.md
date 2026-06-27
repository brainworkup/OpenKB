---
sources: [summaries/SESSION_SUMMARY.md, summaries/QUICK_REFERENCE.md, summaries/AGE_OVERRIDE_GUIDE.md, summaries/README.md, summaries/responses_to_claude.md, summaries/processed_files.md]
brief: Strategies and workflows for storing, organizing, retrieving, and governing clinical files across heterogeneous environments.
---

# Clinical Data Management

Clinical data management encompasses the strategies, workflows, and technical practices used to store, organize, retrieve, and govern clinical files — including neuropsychological assessment PDFs, score exports, and associated reports — across the heterogeneous storage environments typical of academic medical centers and private practices.

## The Problem: Data Sprawl

Clinical data, particularly assessment output files like PAI reports (see [[concepts/pai-assessment]]), tends to accumulate across multiple disjointed locations over time. A concrete example from neuropsychological practice illustrates this well: a single `pai.pdf` file may exist in 208 locations spanning:

- Local archived directories organized by year and patient name
- Active report folders on network drives
- Cloud storage mounts (OneDrive, Dropbox, Google Drive) — sometimes with both a CloudStorage symlink path and a Group Containers path representing the same underlying data
- Knowledge base and reference directories
- Downloads folders and even the system Trash

This kind of sprawl emerges organically as clinicians move files between workflows, create versioned subdirectories for longitudinal cases, and sync data across platforms.

## Consolidation Strategies

A common remediation approach is scripted bulk consolidation — using shell scripts (e.g., `find_and_copy_pai.sh`) to recursively search the filesystem and copy all instances of a target file type into a single canonical directory, renaming files sequentially. This enables downstream processing such as [[concepts/pdf-score-extraction]], [[concepts/ocr-pipeline]], and batch scoring pipelines.

**Key considerations when consolidating:**

- **Deduplication**: Cloud sync tools often mount the same data at multiple paths (e.g., `CloudStorage/OneDrive-...` and `Group Containers/UBF8T346G9.OneDriveStandaloneSuite/...`), producing apparent duplicates that are actually mirrors. True deduplication requires hash comparison, not just filename matching.
- **Versioning awareness**: Patients seen longitudinally may have numbered subdirectories (e.g., `Tianyu/Tianyu2/Tianyu3` or `Lauren/Lauren7/Lauren5/Lauren2`) representing successive assessment sessions. Consolidation scripts should preserve or log provenance so the chronological sequence can be reconstructed.
- **IME vs. clinical distinction**: Independent Medical Examination (IME) cases and standard clinical evaluations may be stored in separate folder hierarchies and should be tagged or separated during consolidation.
- **Trash recovery**: Files deleted to the system Trash may still contain valid clinical data and should be considered during comprehensive consolidation sweeps.

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

## Shared Clinical Reference Materials

Beyond per-patient files, clinical data management also encompasses shared reference resources used across all patient workflows. The `clinical_resources/books/` directory exemplifies this pattern: it holds clinical reference books and textbooks (PDF, DOCX, or text files) that are automatically ingested whenever any patient's RAG system is rebuilt. This centralized approach ensures consistent access to authoritative reference knowledge across multiple patient-specific retrieval systems, and it supports the [[concepts/retrieval-augmented-generation]] architecture used for clinical decision support.

Best practices for this shared layer include:
- Using descriptive filenames for all reference materials
- Storing files in widely supported formats (PDF, DOCX, plain text)
- Reviewing and updating the directory regularly to reflect current clinical standards

## Privacy and Compliance Implications

Because clinical files contain protected health information (PHI), data consolidation scripts must be operated with care. Running such scripts with elevated privileges (`sudo`) on machines that also sync to personal cloud accounts creates risks of PHI co-mingling with non-institutional storage. See [[concepts/phi-data-handling]] and [[concepts/clinical-data-privacy]] for related guidance.

Redaction and de-identification workflows (see [[concepts/pii-redaction-pipelines]]) should be applied before any consolidated files are used in research, ML training, or knowledge base construction. Shared reference materials that do not contain PHI (e.g., textbooks and clinical manuals) carry lower risk but should still be managed under appropriate access controls.

## Relationship to Downstream Pipelines

Consolidated clinical PDFs serve as the input layer for several downstream processes:

- [[concepts/pdf-score-extraction]] — parsing numerical scores from assessment output PDFs
- [[concepts/ocr-pipeline]] — converting scanned or image-based PDFs to machine-readable text
- [[concepts/neuropsychological-assessment-pipeline]] — end-to-end workflows from raw data to structured report
- [[concepts/pai-knowledge-base]] — building a queryable knowledge base from PAI assessment data
- [[concepts/clinical-pdf-assessment]] — systematic review and interpretation of PDF-format clinical instruments
- [[concepts/retrieval-augmented-generation]] — shared reference materials feeding patient-specific RAG systems

## Related Concepts

- [[concepts/pai-assessment]]
- [[concepts/neuropsychological-reporting]]
- [[concepts/phi-data-handling]]
- [[concepts/clinical-data-privacy]]
- [[concepts/pii-redaction-pipelines]]
- [[concepts/pdf-score-extraction]]
- [[concepts/ocr-pipeline]]
- [[concepts/parquet-as-knowledge-store]]
- [[concepts/retrieval-augmented-generation]]
- [[concepts/knowledge-base-architecture]]

## Related Documents

- [[summaries/processed_files]]
- [[summaries/README]]
- [[summaries/AGE_OVERRIDE_GUIDE]]
- [[summaries/QUICK_REFERENCE]]
- [[summaries/SESSION_SUMMARY]]
- [[summaries/responses_to_claude]]