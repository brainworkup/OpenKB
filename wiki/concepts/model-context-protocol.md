---
sources: [summaries/POSITRON_DATABOT_TROUBLESHOOTING.md, summaries/report-generation.md, summaries/mcp-integration.md, summaries/overview.md, summaries/002-mcp-llm-integration.md]
brief: Open standard for exposing AI tools through a provider-agnostic interface, enabling local LLM integration.
---

# Model Context Protocol (MCP)

The **Model Context Protocol (MCP)** is an open standard that defines how applications expose AI-powered tools and capabilities through a uniform, provider-agnostic interface. Rather than hardcoding integrations with specific LLM providers, MCP allows systems to discover, invoke, and swap AI tools without changing application logic.

## Core Idea

MCP acts as a contract layer between an application and its AI backend. An MCP server wraps one or more AI tools (e.g., PDF extraction, text generation, clinical score lookup) and exposes them via a standardized API. The consuming application calls these tools without needing to know which LLM or provider is running underneath.

## Key Characteristics

- **Provider-agnostic**: Works with local models (e.g., Ollama) and cloud providers (e.g., OpenAI, Anthropic)
- **Tool discovery**: Clients can query an MCP server to learn what tools are available
- **Separation of concerns**: Application logic is decoupled from AI infrastructure
- **Swappable backends**: Changing the underlying LLM requires no code changes in the consuming application
- **Standardized tool interfaces**: Each tool has a defined input/output schema

## Architecture

A typical MCP deployment follows a three-tier pattern:

```text
┌─────────────────┐
│  Application    │
└────────┬────────┘
         │ MCP Protocol
┌────────┴────────┐
│  MCP Server     │
│  (Local)        │
└────────┬────────┘
         │ HTTP API
┌────────┴────────┐
│  Ollama         │
│  (LLM Backend)  │
└─────────────────┘
```

The MCP server is restricted to localhost by default, ensuring no external network access and enforcing user-level permissions.

## Usage in the Voice Project

As documented in [[summaries/002-mcp-llm-integration]] and [[summaries/mcp-integration]], the Voice project adopted MCP to power AI capabilities for clinical neuropsychological workflows, including:

- **PDF Extraction**: Extracts structured test score data (subtests, scaled scores, percentiles, demographics) from psychological test PDFs such as WISC-V and WAIS-IV
- **Clinical Interpretation**: Generates narrative interpretations by analyzing score patterns, comparing to normative data, and identifying strengths and weaknesses
- **Lookup Table Integration**: Maps raw scores to standardized clinical terminology via a CSV lookup table
- Automating report section generation
- Processing natural language in clinical contexts

The implementation uses a local [[concepts/local-llm-inference]] backend (Ollama) exposed through MCP, configured via `config.yml`. The default model is `ollama/llama3.1`, reachable at `http://localhost:11434/v1`.

## MCP Tools

### PDF Extraction Tool
Reads PDF content, parses test and subtest scores, extracts demographic information, applies clinical terminology mappings from a lookup table, and outputs structured JSON.

### Clinical Interpretation Tool
Analyzes score patterns, compares against normative data, identifies cognitive strengths and weaknesses, and produces markdown-formatted domain-specific summaries and clinical significance statements.

### Lookup Table Integration
Queries a CSV-based lookup table to map raw test scores to descriptive ranges, clinical labels, and standardized terminology — ensuring consistency across reports.

## Client Integration Pattern

```python
from mcp import Client

client = Client(
    base_url="http://localhost:11434/v1",
    model="ollama/llama3.1"
)

# Invoke a tool
result = client.call_tool(
    "extract_pdf_data",
    {"pdf_path": "data/raw/pdf/wisc5.pdf",
     "output_path": "results/wisc5_report_structure.json"}
)
```

## Why MCP Over Alternatives

| Alternative | Problem |
|---|---|
| Direct OpenAI/Anthropic API calls | Privacy risk, cost, network dependency |
| LangChain | Over-engineering; excessive abstraction |
| Custom LLM integration | High maintenance burden |

MCP was chosen because it provides the flexibility of [[concepts/local-llm-inference]] for [[concepts/privacy-first-software]] while remaining open to cloud backends if needed.

## Performance Considerations

Model selection affects the speed/quality tradeoff:

- **`llama3.1`** — good balance of speed and quality (default)
- **`llama3.1:70b`** — higher quality, slower inference
- **`mistral`** — faster, suitable for simpler tasks

Caching extracted JSON results avoids re-processing already-handled PDFs. Batch processing via `ThreadPoolExecutor` supports parallel PDF handling.

## Error Handling

Common failure modes and resolutions:

- **Connection refused**: Ollama not running — start with `ollama serve`
- **Model error**: Model not downloaded — run `ollama pull llama3.1`
- **Data validation**: Check that extracted JSON contains a non-empty `tests` key before downstream processing

## Relationship to Privacy

Because MCP is backend-agnostic, it enables a fully local deployment stack — a critical requirement when handling sensitive neuropsychological and patient data. All LLM operations run locally: no data leaves the machine, no external API calls are made, and the operator retains full control over model versions. See [[concepts/clinical-data-privacy]] and [[concepts/phi-data-handling]] for the broader privacy considerations this supports.

Recommended practices include sanitizing patient data before processing, using anonymized identifiers, and storing sensitive data securely.

## Configuration

MCP integration is declared in the `mcp` section of `config.yml`. See [[concepts/yaml-configuration]] for patterns used across the project. Key parameters include `pdf_path`, `tree_path`, `llm_base_url`, `llm_model`, and `lookup_table`.

## Related Concepts

- [[concepts/local-llm-inference]] — The local LLM runtime (Ollama) used as the MCP backend
- [[concepts/privacy-first-software]] — Design principle driving the choice of local MCP deployment
- [[concepts/clinical-data-privacy]] — Why patient data must not leave the local environment
- [[concepts/phi-data-handling]] — Regulatory and ethical handling of protected health information
- [[concepts/neuropsychological-assessment-pipeline]] — The clinical workflow MCP tools support
- [[concepts/neuropsychological-test-scores]] — The score data structures extracted and interpreted by MCP tools
- [[concepts/multi-agent-orchestration]] — Broader patterns for composing AI tool calls
- [[concepts/yaml-configuration]] — Configuration patterns used to declare MCP settings

## References

- [MCP Specification](https://modelcontextprotocol.io/)
- [Ollama Documentation](https://ollama.com/)
- [[summaries/002-mcp-llm-integration]] — ADR documenting the adoption of MCP in the Voice project
- [[summaries/mcp-integration]] — Full implementation guide including tools, workflow steps, and troubleshooting

See also: [[summaries/overview]]

See also: [[summaries/report-generation]]

See also: [[summaries/POSITRON_DATABOT_TROUBLESHOOTING]]