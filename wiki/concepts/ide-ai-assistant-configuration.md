---
sources: [summaries/cognition.instructions.md, summaries/brand-and-skills.md, summaries/POSITRON_DATABOT_TROUBLESHOOTING.md]
brief: Configuring AI coding assistants in IDEs via settings files, provider selection, model choice, and instruction files.
---

# IDE AI Assistant Configuration

IDE AI assistant configuration refers to the process of setting up, enabling, and tuning AI-powered coding and data assistants within an integrated development environment. This includes selecting LLM providers, managing API keys, specifying models, integrating local or cloud-based inference backends, and providing project-level instruction files that shape AI behavior.

## Core Components

### 1. Workspace Settings File
Most modern IDEs (including Positron and VS Code) use a `.vscode/settings.json` file to store workspace-level AI assistant configuration. This file controls:
- Which AI providers are enabled
- Which models are selected by default
- How API credentials are sourced
- Runtime interpreter paths (e.g., R, Python)

### 2. AI Instruction Files
Beyond settings, projects can include dedicated instruction files that provide AI assistants with persistent context and behavioral guidelines. A common pattern is a `cognition.instructions` file (or similar `.instructions` file) that:
- Applies globally to all files in the project via a wildcard scope (`applyTo: "**"`)
- Supplies project context so the AI understands the codebase domain
- Defines coding guidelines the AI must follow when generating code, answering questions, or reviewing changes
- Serves as a complement to `settings.json` — settings control *which* AI is active, instruction files control *how* it behaves

This pattern is closely related to [[concepts/documentation-as-code]] and [[concepts/yaml-configuration]], where structured text files encode policy and context rather than just tool settings.

### 3. Provider Configuration
AI assistants typically support multiple LLM providers simultaneously. Common patterns include:
- **Cloud providers**: Anthropic (Claude), OpenAI — require API keys
- **Local providers**: Ollama — require a running local server and installed models
- **Environment-based secrets**: API keys stored as environment variables (`ANTHROPIC_API_KEY`) rather than hardcoded in config files

### 4. Model Selection
Choosing the right model matters for both quality and behavior:
- **Claude Sonnet 4** is the recommended default for Positron Databot
- **Claude 3.5 Sonnet v2** is also supported
- **Claude 3.7 Sonnet** is explicitly discouraged — it performs too many steps before returning control to the user
- Local models (e.g., `llama3:latest` via Ollama) can be registered as custom entries

### 5. Research Preview Acknowledgment
Some AI assistant features (such as Positron Databot) are in experimental or research preview status. Configuration must explicitly acknowledge this:
```json
"databot.researchPreviewAcknowledgment": "Acknowledged"
```

### 6. Runtime Integration
AI assistants in data science IDEs often need to interact with language runtimes. Configuration must specify:
- R interpreter path (e.g., `/usr/bin/R`)
- Python interpreter path (e.g., `.venv/bin/python`)
- Correct Python version in `.python-version`

## Example: Positron Databot Configuration

From [[summaries/POSITRON_DATABOT_TROUBLESHOOTING]], a complete working configuration looks like:

```json
{
  "databot.researchPreviewAcknowledgment": "Acknowledged",
  "positron.assistant.enabled": true,
  "positron.assistant.enabledProviders": ["ollama", "anthropic-api"],
  "positron.assistant.models.custom": {
    "ollama": [{"name": "llama3:latest", "identifier": "llama3:latest"}]
  },
  "positron.languageModels.providers": {
    "anthropic": {"enabled": true, "apiKeySource": "environment"}
  },
  "positron.languageModels.defaultModel": "claude-sonnet-4"
}
```

## Example: Project Instruction File

A minimal `cognition.instructions` file might look like:

```markdown
---
applyTo: "**"
---

Provide project context and coding guidelines that AI should follow when
generating code, answering questions, or reviewing changes.
```

The `applyTo` field is a glob pattern determining which files the instructions apply to. Setting it to `"**"` makes the instructions global across the entire project.

## Common Configuration Pitfalls

| Problem | Impact | Fix |
|---|---|---|
| Missing settings.json | Assistant won't initialize | Create `.vscode/settings.json` |
| No API key set | Cloud providers fail silently | Export `ANTHROPIC_API_KEY` |
| Wrong Python version | Runtime errors | Update `.python-version` to installed version |
| Ollama enabled but no models | Provider error | Run `ollama pull <model>` |
| Missing R runtime | R-based features unavailable | Install R via package manager |
| No instruction file | AI lacks project context | Add a `cognition.instructions` or equivalent file |

## Security Considerations

- Never hardcode API keys in `.vscode/settings.json` — use `"apiKeySource": "environment"` instead
- Treat `.vscode/settings.json` as potentially committed to version control; keep secrets out of it
- Use shell profile files (`~/.zshrc`, `~/.bashrc`) for persistent environment variable storage
- Instruction files (`.instructions`) are generally safe to commit — they contain guidelines, not secrets

## Minimum Version Requirements

Some AI features have strict IDE version requirements. For example, Positron Databot requires **Positron 2025.08.0 or later**. Always check feature documentation before configuring.

## Related Concepts

- [[concepts/positron-ide]] — The Positron IDE that hosts the databot assistant
- [[concepts/local-llm-inference]] — Running models locally via Ollama as an alternative to cloud providers
- [[concepts/openai-compatible-api]] — Standard API interface used by many LLM providers
- [[concepts/yaml-configuration]] — Configuration file patterns used in IDE and tool settings
- [[concepts/retrieval-augmented-generation]] — Underlying technique powering AI assistants like Databot
- [[concepts/r-python-integration]] — Runtime integration required for data science AI assistants
- [[concepts/phi-data-handling]] — Privacy considerations when AI assistants interact with sensitive data
- [[concepts/documentation-as-code]] — The broader practice of encoding policy and context in plain text files

See also: [[summaries/cognition.instructions]], [[summaries/brand-and-skills]]