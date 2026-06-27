---
sources: [summaries/Hermes-Agent-Documentation-Hermes-Agent.md]
brief: Deploying AI agents across 20+ messaging platforms from a single unified gateway interface.
---

# Multi-Platform Messaging for AI Agents

Multi-platform messaging refers to the capability of an AI agent to communicate with users and deliver outputs across many different messaging and collaboration platforms — all from a single, unified backend. Rather than building separate integrations for each service, a multi-platform approach allows one agent deployment to be accessible wherever users already spend their time.

## Why It Matters

A core limitation of traditional AI assistants is that they are tied to a specific interface — a web chat window, an IDE plugin, or a proprietary app. Multi-platform messaging dissolves this constraint: the agent lives where the user is, not the other way around. This is especially important for autonomous agents that run continuously in the background (e.g., on a remote VPS or serverless infrastructure), since the user needs an ambient, low-friction way to interact with and receive results from the agent regardless of what device or app they are using at any given moment.

## Hermes Agent Implementation

As documented in [[summaries/Hermes-Agent-Documentation-Hermes-Agent]], Hermes Agent supports **20+ platforms** from a single gateway. These include:

### Consumer Messaging
- Telegram
- WhatsApp
- Signal
- WeChat (Weixin)
- QQ Bot
- Yuanbao
- BlueBubbles (iMessage bridge)
- DingTalk
- Feishu
- WeCom

### Team Collaboration
- Slack
- Discord
- Microsoft Teams
- Google Chat
- Mattermost
- Matrix

### Other Channels
- Email
- SMS
- Home Assistant
- CLI (command-line interface)

## Integration with Agent Capabilities

Multi-platform messaging is not just a delivery mechanism — it is tightly coupled with other agent features:

- **Scheduled automations:** Built-in cron jobs can deliver results to any supported platform, meaning a scheduled research task can push its summary to Telegram or Slack automatically.
- **Voice interaction:** Platforms like Telegram, Discord, and Discord Voice Channels support real-time voice interaction, extending the messaging layer into audio.
- **Remote operation:** Because Hermes can run on cloud infrastructure (a $5 VPS, Daytona, Modal), a user can communicate with it entirely through Telegram while the agent performs work on a remote machine the user never directly accesses.

## Relationship to Persistent Memory

Multi-platform messaging intersects meaningfully with [[concepts/persistent-memory]]. A user might interact with the agent across different platforms on different days — via Telegram on mobile, via CLI at a workstation, via Slack at work. The agent's cross-session memory and user modeling capabilities ensure continuity of context regardless of which channel the conversation arrives through.

## Design Principle

The underlying design principle is **channel agnosticism**: the agent's intelligence, memory, and tool access are decoupled from the communication channel. The platform is merely a transport layer. This makes the agent robust to changes in user habits, platform availability, and organizational tooling.

## See Also
- [[summaries/Hermes-Agent-Documentation-Hermes-Agent]] — Full overview of Hermes Agent, including its platform list and architecture
- [[concepts/persistent-memory]] — How the agent maintains context across sessions and platforms
