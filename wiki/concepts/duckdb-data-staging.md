---
sources: [summaries/copilot-instructions.md, summaries/LLM_AGENT_MAP.md, summaries/README.md]
brief: Using DuckDB as a high-performance data staging layer for neuropsychological test score pipelines.
---

# DuckDB Data Staging for Clinical Pipelines

DuckDB data staging refers to the use of DuckDB—an embedded, columnar analytical database—as the central data layer for ingesting, caching, querying, and transforming raw neuropsychological test scores before they reach downstream processing stages such as LLM narrative generation, visualization, and report rendering.

## Core Concept

In clinical neuropsychological workflows, raw score data typically arrives as CSV exports from test administration platforms. A DuckDB-backed staging layer converts these into optimized columnar formats (Parquet) and loads them into an in-process DuckDB database, enabling fast analytical queries without requiring a separate server process.

The canonical pipeline is:

```
CSV (raw scores) → Parquet (columnar storage) → DuckDB (query engine)
```

This approach is central to the [[concepts/neuropsychological-assessment-pipeline]] and is implemented in the [[summaries/README]] (`cingulate` R package) as its primary data infrastructure.

## Why DuckDB for Clinical Data

- **Embedded operation**: No server setup; the database runs in-process within R or Python, which fits [[concepts/local-first-architecture]] deployments where PHI cannot leave the local machine.
- **Columnar analytics**: DuckDB is optimized for aggregations and filtering over large tabular datasets—ideal for querying scores across cognitive domains, age groups, or patient cohorts.
- **Parquet interoperability**: DuckDB reads and writes Parquet natively, enabling efficient caching and sharing of processed data without re-running expensive score transformations.
- **R and Python integration**: DuckDB has first-class bindings for both R and Python, supporting the [[concepts/r-python-integration]] pattern used in the `cingulate` pipeline.

## Role in the cingulate Pipeline

In the [[summaries/README]] architecture, DuckDB staging serves several functions:

1. **Ingestion**: Raw CSV score files are loaded and validated.
2. **Transformation**: Scores are normalized, domain-tagged, and stored as [[concepts/long-format-clinical-data]] records.
3. **Caching**: Intermediate results (e.g., computed percentiles, z-scores) are persisted in Parquet/DuckDB to avoid redundant computation.
4. **Domain querying**: [[concepts/domain-processor-pattern]] R6 classes query DuckDB to retrieve domain-specific score sets (IQ, memory, attention, language, etc.) for LLM prompting and report generation.

## Configuration

A basic DuckDB connection in R looks like:

```r
library(duckdb)
con <- dbConnect(duckdb::duckdb(), dbdir = "path/to/database.db")
dbListTables(con)
```

The `cingulate` package uses the **duckdb** R package alongside **dplyr**, **here**, and other tidyverse-adjacent tools to manage these connections within its R6 class infrastructure.

## Relationship to Vector Storage

Beyond staging tabular scores, DuckDB can also serve as a lightweight vector store for embeddings. See [[concepts/duckdb-as-vector-store]] for details on that pattern, which complements the staging use case by storing semantic representations of clinical text alongside structured score data.

## Related Parquet Pattern

The intermediate Parquet layer is discussed separately in [[concepts/parquet-as-knowledge-store]], which covers how Parquet files act as durable, portable snapshots of processed clinical data that can be versioned and audited.

## Privacy and PHI Considerations

Because DuckDB is embedded and file-based, all data remains on-premises by default. This is a key advantage for [[concepts/phi-data-handling]] and [[concepts/clinical-data-privacy]] in neuropsychological contexts where patient identifiers and cognitive scores constitute protected health information.

## Related Concepts

- [[concepts/neuropsychological-assessment-pipeline]]
- [[concepts/neuropsychological-assessment-automation]]
- [[concepts/domain-processor-pattern]]
- [[concepts/long-format-clinical-data]]
- [[concepts/duckdb-as-vector-store]]
- [[concepts/parquet-as-knowledge-store]]
- [[concepts/local-first-architecture]]
- [[concepts/r6-class-architecture]]
- [[concepts/phi-data-handling]]
- [[concepts/clinical-data-privacy]]
- [[concepts/r-python-integration]]


See also: [[summaries/LLM_AGENT_MAP]]

See also: [[summaries/copilot-instructions]]