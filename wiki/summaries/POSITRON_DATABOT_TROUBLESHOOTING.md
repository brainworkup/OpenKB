---
doc_type: short
full_text: sources/POSITRON_DATABOT_TROUBLESHOOTING.md
---

# Positron Databot Troubleshooting Guide

## Overview

This document is a practical troubleshooting guide for setting up the **Positron Databot** extension in a repository. It identifies five key issues preventing the databot from running and provides fixes for each, along with the required configuration and verification steps.

## Key Issues Identified

### 1. Missing Workspace Configuration ✅ (Fixed)
- No `.vscode/settings.json` existed.
- Fixed by creating the file with databot-specific settings: research preview acknowledgment, Positron Assistant enabled, Anthropic provider, and Claude Sonnet 4 as default model.

### 2. Missing R Runtime ❌ (Critical)
- R is not installed or not on the system PATH.
- Required because this is an **R-based PAI RAG system**.
- Install via:
  - macOS: `brew install r`
  - Ubuntu/Debian: `sudo apt-get install r-base r-base-dev`
  - Windows: Download from [CRAN](https://cran.r-project.org/)
- Verify with `R --version`.

### 3. Python Version Mismatch ⚠️
- `.python-version` specifies Python 3.14 (nonexistent).
- System has Python 3.11.14 and 3.12 available.
- Fix: update `.python-version` to `3.12` or `3.11`.

### 4. Missing Anthropic API Key ⚠️
- Databot requires an Anthropic API key.
- Set via environment variable: `export ANTHROPIC_API_KEY="your-api-key-here"`
- Can also be configured in [[concepts/positron-ide]] Settings under "Language Model Providers".

### 5. Ollama Provider Has No Models ❌
- Error: `[Ollama] No models available for provider`.
- Fix: install a model (`ollama pull llama3`) and add a custom model entry in `.vscode/settings.json`.
- Related to [[concepts/local-llm-inference]] — running models locally via Ollama requires at least one model to be pulled and registered.

## Positron Databot Requirements

- **Minimum Version**: Positron 2025.08.0+
- **Research Preview Acknowledgment**: Required
- **Positron Assistant**: Must be enabled
- **API Key**: Anthropic API key
- **Recommended Model**: Claude Sonnet 4 or Claude 3.5 Sonnet v2
- ⚠️ Claude 3.7 Sonnet is **not recommended** (does too much work before returning control)

## Workspace Configuration (`.vscode/settings.json`)

Key settings include:
- `databot.researchPreviewAcknowledgment`: `"Acknowledged"`
- `positron.assistant.enabled`: `true`
- Enabled providers: `ollama`, `anthropic-api`
- Custom Ollama model: `llama3:latest`
- Anthropic API key sourced from environment
- Default model: `claude-sonnet-4`
- R interpreter path: `/usr/bin/R`
- Python interpreter path: `${workspaceFolder}/.venv/bin/python`

See [[concepts/ide-ai-assistant-configuration]] for general patterns around configuring AI assistants in development environments, and [[concepts/yaml-configuration]] for how settings files like this are structured.

## Verification Steps

1. Restart Positron
2. Open an R or Python file
3. Check for the databot icon in the interface
4. Ask databot a simple question

## Next Steps (Priority Order)

1. Install R (critical)
2. Fix Python version in `.python-version`
3. Set `ANTHROPIC_API_KEY` environment variable
4. Restart Positron
5. Verify databot accessibility

## Related Concepts
- [[concepts/model-context-protocol]]
- [[concepts/pai-assessment]]
- [[concepts/pai-knowledge-base]]
- [[concepts/python-project-structure]]

- [[concepts/positron-ide]] — The Positron IDE in which the databot extension runs
- [[concepts/ide-ai-assistant-configuration]] — Configuring AI assistants within development environments
- [[concepts/local-llm-inference]] — Running local LLMs with Ollama
- [[concepts/r-python-integration]] — Runtime environment coordination for R and Python projects
- [[concepts/retrieval-augmented-generation]] — The RAG architecture underlying the PAI RAG system this databot supports
- [[concepts/openai-compatible-api]] — API compatibility patterns relevant to provider configuration