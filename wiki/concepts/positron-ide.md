---
sources: [summaries/copilot-instructions.md, summaries/POSITRON_DATABOT_TROUBLESHOOTING.md]
brief: Positron is a data science IDE by Posit with built-in AI assistant and multilingual R/Python support.
---

# Positron IDE

Positron is a next-generation data science integrated development environment (IDE) developed by [Posit](https://posit.co) (formerly RStudio). It is built on VS Code's foundation and designed to support both R and Python workflows natively, with deep integration for AI-assisted coding via the **Positron Assistant**.

## Key Features

### Multilingual Runtime Support
Positron provides first-class support for both R and Python interpreters, configurable via workspace settings. This makes it particularly well-suited for projects that blend R and Python — such as [[concepts/r-python-integration]] workflows — without requiring separate IDE setups.

### Positron Assistant (AI Integration)
Positron includes a built-in AI assistant that can be configured with multiple large language model providers. Configuration is managed through [[concepts/ide-ai-assistant-configuration]] and [[concepts/yaml-configuration]] in `.vscode/settings.json`.

Supported providers include:
- **Anthropic** (Claude models — recommended)
- **Ollama** (local models via [[concepts/local-llm-inference]])
- Other OpenAI-compatible endpoints

### Databot Extension
Databot is a research-preview feature (as of Positron 2025.08.0+) that enables AI-powered assistance specifically within the data science context — helping users understand code, query data, and get contextual suggestions. It requires:
- Research preview acknowledgment in settings
- Positron Assistant enabled
- A configured LLM provider (Anthropic API key recommended)
- R and/or Python runtimes installed

See [[summaries/POSITRON_DATABOT_TROUBLESHOOTING]] for a detailed breakdown of common setup issues.

## Configuration

Positron workspace configuration lives in `.vscode/settings.json`. Key settings for AI and databot functionality:

```json
{
  "databot.researchPreviewAcknowledgment": "Acknowledged",
  "positron.assistant.enabled": true,
  "positron.languageModels.defaultModel": "claude-sonnet-4",
  "positron.interpreter.r": { "enabled": true, "path": "/usr/bin/R" },
  "positron.interpreter.python": { "enabled": true, "path": ".venv/bin/python" }
}
```

## Model Recommendations

| Model | Status |
|---|---|
| Claude Sonnet 4 | ✅ Recommended |
| Claude 3.5 Sonnet v2 | ✅ Supported |
| Claude 3.7 Sonnet | ⚠️ Not recommended (over-generates before returning control) |
| Ollama local models | ✅ Supported with manual model config |

## Common Setup Issues

1. **Missing R runtime** — R must be installed and on PATH for R-based projects
2. **Python version mismatch** — `.python-version` must reference an installed Python version
3. **Missing Anthropic API key** — Set `ANTHROPIC_API_KEY` as an environment variable
4. **Ollama no models error** — Must pull at least one model with `ollama pull <model>`
5. **Missing workspace config** — `.vscode/settings.json` must exist with proper settings

## Related Concepts

- [[concepts/ide-ai-assistant-configuration]] — Configuring LLM providers within IDE settings
- [[concepts/local-llm-inference]] — Running models like Ollama locally
- [[concepts/r-python-integration]] — Mixed R and Python workflows
- [[concepts/retrieval-augmented-generation]] — RAG pipelines that databot may interact with
- [[concepts/yaml-configuration]] — Settings file structure and management
- [[concepts/openai-compatible-api]] — API compatibility layer used by some Positron providers

## References

- [Positron Databot Documentation](https://positron.posit.co/databot.html)
- [Introducing Databot — Posit Blog](https://posit.co/blog/introducing-databot/)
- [[summaries/POSITRON_DATABOT_TROUBLESHOOTING]]


See also: [[summaries/copilot-instructions]]