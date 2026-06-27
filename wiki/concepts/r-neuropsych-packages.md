---
sources: [summaries/Apply-to-Y-Combinator-JWT.md, summaries/DEPENDENCIES.md]
brief: R packages supporting Luria’s neuropsych analysis and reporting stack.
---

# R Packages for Neuropsychological Analysis

The Luria workspace uses a curated set of R packages alongside its Python stack to perform psychometric scoring, statistical modeling, and report generation. In the YC application summary, the founder describes the system’s origin as starting in R for real clinical workload relief: first with basic statistical programming, then with functions to extract data from CSVs, PDFs, and eventually broader file types, before expanding into today’s larger Luria system. In that sense, these packages are not peripheral utilities; they represent part of the original substrate of the [[concepts/luria-neuropsych-pipeline]] and continue to complement the [[concepts/r-python-integration]] layer.

See [[summaries/DEPENDENCIES]] for the full dependency manifest.

---

## Core Tidyverse Packages

The tidyverse forms the data-wrangling backbone:

- **`dplyr`** — data manipulation for filtering, grouping, summarizing, and transforming score tables and extracted clinical variables
- **`tidyr`** — reshaping between wide and long formats; critical for [[concepts/long-format-clinical-data]] workflows
- **`ggplot2`** — cognitive profile plots and statistical visualization; theming is coordinated via [[concepts/r-visualization-theming]]

Within Luria’s evolution, these packages supported the transition from ad hoc manual processing to reproducible neuropsychological analysis pipelines, especially when working across heterogeneous source materials and structured score exports.

---

## Psychometric & Statistical Modeling

- **`psych`** — descriptive statistics tailored to psychometric data: reliability coefficients, score distributions, scale summaries, and factor-oriented exploration. Central to [[concepts/neuropsychological-test-scores]] workflows.
- **`lavaan`** — structural equation modeling (SEM) and confirmatory factor analysis (CFA). Used to validate latent cognitive domain structures aligned with [[concepts/cognitive-domains]] theory.

These packages reflect the founder’s dual researcher-clinician background described in [[summaries/Apply-to-Y-Combinator-JWT]], where scientific rigor and clinical interpretation are both core to the product’s design. They help preserve neuropsychological reasoning rather than reducing report generation to generic text automation.

---

## Reporting & Document Generation

- **`knitr`** — dynamic report engine; executes R code chunks embedded in R Markdown documents
- **`rmarkdown`** — generates HTML, PDF, and DOCX outputs from literate programming source files

Historically, report writing in the project began with R and R Markdown before later expanding to LaTeX, [[concepts/quarto]], and Typst-based rendering. This makes the R reporting stack important both as a legacy foundation and as an ongoing option for modular clinical output generation.

These packages support the [[concepts/narrative-report-generation]] pipeline when reports are authored or post-processed on the R side, especially in workflows where quantitative results must be converted into clinically interpretable narrative structure.

---

## Data I/O & Database Connectivity

- **`jsonlite`** — reads extracted JSON records produced by the Python extraction subagents
- **`DBI`** — generic database interface abstraction
- **`RSQLite`** — SQLite driver; reads shared structured stores directly from R, enabling cross-language access to the same [[concepts/clinical-data-management]] layer

This role fits the broader Luria architecture described in the YC application: a system that grew from extracting data out of CSV and PDF sources into a more comprehensive workflow that can organize, analyze, and synthesize neuropsychological information across steps of care.

---

## Python Interoperability

- **`reticulate`** — embeds a Python session inside R, enabling direct calls to Luria Python modules. This is the foundation of [[concepts/r-python-integration]] within the workspace, allowing R scripts to invoke Python extraction results or ML classifiers without data export/import round-trips.

In practice, this interoperability helps preserve earlier R-based clinical and research logic while integrating it into a newer multi-component system. It supports continuity between the founder’s original R tooling and the current Luria environment, which is now more agentic and automation-oriented overall.

---

## Internal Package

- **`cingulate`** — a custom internal R package located at `agent/cingulate/`. Provides neuropsychological analysis functions specific to the Luria/brainworkup context. See [[concepts/cingulate-engine]] for architectural details.

The existence of an internal package underscores that the R environment is not only for general-purpose statistics, but also for domain-specific neuropsychological operations shaped by real pediatric, clinical, and forensic evaluation needs.

---

## Relationship to the Broader Stack

R packages occupy a specialized niche within the Luria architecture:

| Layer | R Role |
|---|---|
| Data wrangling | `dplyr`, `tidyr` post-process score exports and extracted variables |
| Psychometrics | `psych`, `lavaan` validate score structure and domain models |
| Reporting | `knitr`, `rmarkdown` render supplementary or legacy clinical outputs |
| Storage access | `RSQLite`, `DBI` read shared structured data stores |
| Python bridge | `reticulate` enables unified analysis sessions |

The YC application makes clear that Luria’s current vision is a local-first, workflow-wide neuropsychological system that can automate much of evaluation and reporting while preserving clinical quality and privacy. Within that broader stack, R remains especially valuable for psychometrics, reproducible reporting, and the historically important functions from which the system originally grew.

The R environment is invoked via the `R ≥4.3` binary, keeping it consistent with the [[concepts/local-first-architecture]] and [[concepts/privacy-first-software]] orientation of the overall system.

---

## Related Concepts

- [[concepts/r-neuropsych-packages]]
- [[concepts/r-python-integration]]
- [[concepts/r-visualization-theming]]
- [[concepts/cognitive-domains]]
- [[concepts/neuropsychological-test-scores]]
- [[concepts/cingulate-engine]]
- [[concepts/long-format-clinical-data]]
- [[concepts/narrative-report-generation]]
- [[concepts/quarto]]
- [[concepts/clinical-data-management]]
- [[concepts/luria-neuropsych-pipeline]]
- [[concepts/local-first-architecture]]
- [[concepts/privacy-first-software]]
- [[summaries/Apply-to-Y-Combinator-JWT]]