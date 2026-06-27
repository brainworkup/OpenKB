---
sources: [summaries/Luria_AI_Q4_Investor_Memo_2026.md, summaries/bwu.neuro.reports.recs.conversion.md, summaries/bwu.neuro.reports.recs.adhd.older-adult.md, summaries/bwu.neuro.reports.recs.adhd.books.md, summaries/bwu.neuro.reports.recs.adhd.adult.md, summaries/autism_recommendations_for_adults_summary.md, summaries/README.md]
brief: Shared professional practice guidelines that inform clinical AI systems and neuropsychological workflows.
---

# Clinical Guidelines

Clinical guidelines are authoritative, professionally vetted reference materials that establish standards for clinical practice. In the context of AI-assisted clinical workflows, they serve as a foundational knowledge layer that informs [[concepts/retrieval-augmented-generation|retrieval-augmented generation]] systems, narrative generation, and clinical decision support.

## Role in Clinical AI Systems

Clinical guidelines are not generated dynamically — they are curated, stable documents (PDFs, DOCX, or plain text) that are ingested into [[concepts/knowledge-base-architecture|knowledge bases]] and made available to patient-facing AI pipelines. By centralizing these materials in a shared directory, all patient RAG systems can draw from the same authoritative sources, ensuring consistency across evaluations and reports.

This approach supports:
- **Consistency**: Every patient system references the same professional standards.
- **Maintainability**: Updating a guideline in one place propagates automatically on the next system rebuild.
- **Transparency**: Clinicians can audit which guidelines are informing AI-generated content.

## Integration with RAG Pipelines

In systems like those described in [[summaries/AUTISM_RAG_SYSTEM_DOCUMENTATION]] and [[summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_PIPELINE]], clinical guidelines are ingested alongside patient-specific data. The [[concepts/retrieval-augmented-generation|RAG]] pipeline retrieves relevant guideline passages at query time, grounding generated clinical narratives in evidence-based standards.

Related ingestion and retrieval mechanisms include:
- [[concepts/report-ingestion-pipeline]] — handles structured document ingestion
- [[concepts/hybrid-search-retrieval]] — combines semantic and keyword search over guideline content
- [[concepts/rag-chunking]] — governs how long guideline documents are split for retrieval

## Relationship to Clinical Workflows

Clinical guidelines intersect with several downstream processes:

- [[concepts/clinical-narrative-generation]] — narratives should reflect current professional standards
- [[concepts/neuropsychological-reporting]] — reports cite or implicitly rely on diagnostic criteria and best practices
- [[concepts/dsm5-diagnosis-normalization]] — diagnostic language is standardized against recognized classification systems
- [[concepts/clinical-report-structure]] — report organization often follows guideline-recommended formats
- [[concepts/clinical-data-privacy]] — guidelines may include requirements for PHI handling and documentation

## File Management Conventions

In the `clinical_resources/guidelines` directory:
- Supported formats: **PDF**, **DOCX**, plain text
- Files should use **descriptive filenames** to facilitate automated ingestion and retrieval
- Files are automatically picked up when any patient RAG system is rebuilt — no manual linking is required

## See Also

- [[concepts/clinical-data-management]]
- [[concepts/neuropsychological-assessment-pipeline]]
- [[concepts/per-patient-workspace]]
- [[concepts/phi-data-handling]]

## Related Documents
- [[summaries/README]]


See also: [[summaries/autism_recommendations_for_adults_summary]]

See also: [[summaries/bwu.neuro.reports.recs.adhd.adult]]

See also: [[summaries/bwu.neuro.reports.recs.adhd.books]]

See also: [[summaries/bwu.neuro.reports.recs.adhd.older-adult]]

See also: [[summaries/bwu.neuro.reports.recs.conversion]]

See also: [[summaries/Luria_AI_Q4_Investor_Memo_2026]]