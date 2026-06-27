---
doc_type: short
full_text: sources/002-mcp-llm-integration.md
---

# ADR 002: MCP Server Integration for LLM Operations

## Overview

This Architecture Decision Record documents the decision to integrate **Model Context Protocol (MCP)** servers as the standardized interface for AI capabilities in the Voice project, using a local [[concepts/local-llm-inference]] backend for privacy-preserving LLM operations in clinical contexts.

## Problem Statement

The Voice project needs AI-powered capabilities for:
- Extracting structured data from PDF psychological test reports
- Generating clinical interpretations from test scores
- Automating report section generation
- Processing natural language in clinical contexts

## Decision

Use [[concepts/model-context-protocol]] (MCP) servers backed by a local LLM runtime (Ollama).

### Key Architectural Choices

| Concern | Choice | Rationale |
|---|---|---|
| AI interface | MCP servers | Standardized, swappable, tool-discoverable |
| LLM backend | Ollama (local) | Privacy, no API cost, offline support |
| Default model | `ollama/llama3.1` | Balance of capability and local resource use |
| Configuration | `config.yml` `mcp` section | Centralized, environment-agnostic |

## Rationale

### Why MCP
- Standardized protocol enabling LLM tool integration without provider lock-in
- Supports both local and cloud AI providers
- Clean separation of application logic from AI capabilities
- Built-in tool discovery and management

### Why Local LLM (Ollama)
- **Patient data privacy**: Data never leaves the local environment — critical for clinical use
- No recurring API costs
- Customizable models for clinical/psychological terminology
- Offline capability
- Full control over model versions

## Alternatives Rejected

- **Direct OpenAI/Anthropic API**: Privacy concerns, cost, network dependency
- **LangChain**: Over-engineering; unnecessary complexity for this use case
- **Custom LLM integration**: High maintenance burden

## Consequences

**Positive:**
- Privacy-preserving AI operations (suitable for [[concepts/clinical-data-privacy]] and [[concepts/phi-data-handling]])
- Flexible model switching without code changes
- Standardized tool interfaces

**Negative:**
- Requires local GPU/CPU resources
- Initial setup complexity
- Model quality is hardware-dependent

## Implementation Details

- MCP config under `mcp` section of `config.yml` (see [[concepts/yaml-configuration]])
- Ollama endpoint: `http://localhost:11434/v1`
- Default model: `ollama/llama3.1`
- Tools: PDF extraction for psychological reports, lookup table integration for clinical terminology

## References
- [MCP Specification](https://modelcontextprotocol.io/)
- [Ollama Documentation](https://ollama.com/)

## Related Concepts
- [[concepts/privacy-first-software]]
- [[concepts/neuropsychological-assessment-pipeline]]
- [[concepts/neuropsychological-reporting]]
- [[concepts/clinical-nlp-pipelines]]
- [[concepts/pii-redaction-pipelines]]
- [[concepts/retrieval-augmented-generation]]
