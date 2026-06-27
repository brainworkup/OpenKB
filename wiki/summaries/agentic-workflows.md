---
doc_type: short
full_text: sources/agentic-workflows.md
---

# Agentic Workflows for Data Science

Agentic workflows use a central LLM-based orchestrator plus specialized sub-agents or tools to automate multi-step data science pipelines end to end. The document frames this as a way to reduce manual coordination across ingestion, processing, redaction, analysis, and reporting tasks, especially in research or clinical settings.

## Core Idea

A top-level agent acts as planner, dispatcher, and integrator. Given a high-level objective such as generating a neuropsychology report, it:

- decomposes the goal into subtasks
- routes each subtask to an appropriate tool or sub-agent
- monitors execution and retries or handles failures
- assembles outputs into a final deliverable

This is an instance of an [[concepts/orchestrator-worker-pattern]] and relates broadly to [[concepts/multi-agent-orchestration]].

## Pipeline Stages

Using the motivating example from wm, the workflow spans several major stages:

- **Data ingestion**: loading raw data from files, databases, or APIs
- **Data processing**: cleaning, transforming, and structuring data
- **PII redaction**: identifying and removing sensitive personal information
- **Report generation**: producing narrative research or clinical reports
- **Table creation**: formatting structured outputs for review
- **Figure generation**: creating charts and visualizations

Together these form a view of end-to-end automation for data science tasks.

## Hybrid Python and R Environments

A major practical concern is cross-language orchestration. In the `luria` / `cingulate` example, the overall system is Python-based while significant analytical logic lives in R. An agentic pipeline therefore needs to support:

- invoking R from Python, such as through `rpy2` or subprocess execution
- passing data reliably across runtime boundaries
- handling errors, retries, and state consistently when multiple languages are involved

This connects closely to [[concepts/r-python-integration]] and to architecture concerns around tool execution and workflow coordination.

## Architecture Patterns

The document highlights several recurring design patterns:

- **Orchestrator-worker**: a top-level planner delegates to specialized executors
- **Tool-using agents**: agents can call functions, APIs, file operations, or code runtimes
- **Retrieval-augmented generation**: agents retrieve relevant context before acting or writing; see [[concepts/retrieval-augmented-generation]]
- **Memory and state**: workflows preserve intermediate context across long-running tasks

These patterns connect the document to [[concepts/neuropsychological-assessment-workflow]] and broader [[concepts/multi-agent-orchestration]].

## Autonomy and Human Review

The document distinguishes between low-risk steps that can be executed silently and high-risk steps that should involve human oversight.

- deterministic or routine tasks can often run without interruption
- sensitive steps, especially PII decisions or final report approval, should be surfaced for review

This links to [[concepts/silent-operation]]. It also suggests a practical risk-based model of delegation rather than full unsupervised automation.

## Relevance to Sensitive Domains

Because the motivating use case includes clinical reporting, the workflow has implications for privacy, safety, and deployment constraints. This makes [[concepts/local-llm-inference]] relevant where sensitive data should remain on local infrastructure.

## Related Pages

- wm
- cli.brew cask
- cli.ia
- cli.kill
- [[concepts/neuropsychological-assessment-workflow]]
- [[concepts/multi-agent-orchestration]]
- [[concepts/retrieval-augmented-generation]]
- [[concepts/silent-operation]]
- [[concepts/r-python-integration]]
- [[concepts/local-llm-inference]]

## Main Takeaway

The document presents agentic workflows as a practical architecture for automating complex data science pipelines, with particular emphasis on orchestrated multi-stage execution, cross-language integration, and selective autonomy in sensitive clinical or research contexts.

## Related Concepts
- [[concepts/neuropsychological-assessment-automation]]
- [[concepts/pii-redaction-pipelines]]
- [[concepts/agent-pipeline-state-management]]
- [[concepts/langgraph-agent-workflows]]
- [[concepts/luria-neuropsych-pipeline]]
- [[concepts/clinical-data-privacy]]
- [[concepts/local-first-architecture]]
- [[concepts/subagent-architecture]]
- [[concepts/clinical-narrative-generation]]
- [[concepts/report-review-qa]]
