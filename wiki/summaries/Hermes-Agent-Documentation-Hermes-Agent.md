---
doc_type: short
full_text: sources/Hermes-Agent-Documentation-Hermes-Agent.md
---

# Hermes Agent — Documentation Overview

**Source:** Hermes-Agent-Documentation-Hermes-Agent

## What Is Hermes Agent?

Hermes Agent is an **autonomous AI agent** built by [Nous Research](https://nousresearch.com). Its defining characteristic is a **closed learning loop**: it creates skills from experience, self-improves those skills during use, nudges itself to persist knowledge, and builds a deepening model of the user across sessions. It is not a coding copilot or a simple chatbot wrapper — it is designed to become more capable the longer it runs.

## Installation

- **Windows / macOS:** Download the Hermes Desktop installer from the official website.
- **Linux / macOS / WSL2 / Android (Termux):** `curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash`
- **Windows (native, PowerShell):** `iex (irm https://hermes-agent.nousresearch.com/install.ps1)`
- After install, run `hermes setup --portal` to authenticate via OAuth (covers model access + Tool Gateway: web search, image generation, TTS, browser).

## Key Features

### Closed Learning Loop
- Agent-curated persistent memory with periodic nudges
- Autonomous skill creation and self-improvement of skills during use
- FTS5 cross-session recall with LLM summarization
- [Honcho](https://github.com/plastic-labs/honcho) dialectic user modeling across sessions

### Runs Anywhere
- 6 terminal backends: local, Docker, SSH, Daytona, Singularity, Modal
- Serverless persistence via Daytona and Modal (near-zero cost when idle)
- Can run on a $5 VPS, a GPU cluster, or serverless infrastructure

### [[concepts/multi-platform-messaging]] — 20+ Platforms
- CLI, Telegram, Discord, Slack, WhatsApp, Signal, Matrix, Mattermost, Email, SMS, DingTalk, Feishu, WeCom, Weixin, QQ Bot, Yuanbao, BlueBubbles, Home Assistant, Microsoft Teams, Google Chat, and more

### Tools & Integrations
- 60+ built-in tools
- Full web control: search, extract, browse, vision, image generation, TTS via Nous Portal
- MCP support: Connect to any MCP server for extended capabilities
- Programmatic Tool Calling via `execute_code` for collapsing multi-step pipelines

### Open Standard Skills
- Compatible with [agentskills.io](https://agentskills.io)
- Skills are portable, shareable, and community-contributed via the Skills Hub

### Scheduling & Parallelism
- Built-in cron for scheduled automations, with delivery to any platform
- Spawn isolated subagents for parallel workstreams

### Research & Training
- Batch processing, trajectory export, RL training with Atropos
- Built by the lab behind Hermes, Nomos, and Psyche models

### LLM-Friendly Documentation
- `/llms.txt` — curated index (~17 KB)
- `/llms-full.txt` — full docs concatenated (~1.8 MB)

## Model & Provider Support
- Works with Nous Portal, OpenRouter, OpenAI, or any compatible endpoint

## Notable Concepts for Further Exploration
- [[concepts/persistent-memory]]
- [[concepts/multi-platform-messaging]]
- Closed learning loop
- Skill creation and self-improvement
- User modeling across sessions
- MCP server integration