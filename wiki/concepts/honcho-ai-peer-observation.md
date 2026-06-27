---
sources: [summaries/README.md]
brief: A session-based AI pattern where one agent observes and learns from another's user interactions.
---

# Honcho AI Peer Observation Pattern

The **Honcho AI Peer Observation Pattern** is an architectural approach where one AI agent is configured to observe the interactions of another agent within the same session workspace, enabling cross-agent learning, preference modeling, and contextual awareness without direct data sharing pipelines.

See it in use in [[summaries/README]] (Luria Streamlit App).

## Core Concept

Honcho (https://honcho.dev) provides a session and workspace abstraction that allows multiple "peers" to be registered within a shared context. One peer can be flagged with `observe_me: True`, which signals to the platform that its interactions should be made available for other peers to query and reason over. This creates an asymmetric observation relationship — the observed agent acts normally, while the observing agent builds a model of user preferences, behavior, and context over time.

This pattern is particularly relevant in clinical or specialized assistant contexts where one agent handles direct user interaction and another maintains a longitudinal model of the user's needs, communication style, or clinical context.

## How It Works

From [[summaries/README]]:

```python
from honcho import Honcho
honcho = Honcho(api_key="YOUR_API_KEY")
session = honcho.sessions.create(workspace_id="luria-app")
honcho.sessions.add_peers("luria-app", session.id, {"joey": {"observe_me": True}})
response = honcho.peers.chat("luria-app", "joey", query="What does this user like?")
```

Key steps:
1. **Create a workspace** — logical namespace grouping all sessions for an application
2. **Create a session** — a bounded interaction context
3. **Register peers** — named agents within the session; `observe_me: True` marks the observed agent
4. **Peer chat** — an observing agent can query the workspace to synthesize what it has learned about the user

## Relationship to Multi-Agent Orchestration

This pattern is a lightweight alternative to full [[concepts/multi-agent-orchestration]]. Rather than coordinating task execution across agents, Honcho peer observation focuses on **memory and preference accumulation** — one agent learns from another's interactions passively. This complements [[concepts/persistent-memory]] architectures by externalizing the memory store to a hosted service rather than requiring local state management.

## Use in the Luria App

In the [[concepts/luria-neuropsych-pipeline]] context, Honcho is used optionally via [`honcho-luria-app.py`](honcho-luria-app.py). The pattern allows a secondary observer agent to build a model of the clinician's preferences, question patterns, and reporting style — without the primary clinical assistant needing to change its behavior. This is a non-intrusive way to accumulate [[concepts/style-profiles]] and [[concepts/knowledge-continuity]] about individual clinicians over time.

This integrates naturally with the Luria Voice "SOUL" layer (see [[summaries/0008-soul-single-file-style-agent-architecture]]), where a clinician's style is captured and injected into report generation.

## Relation to Privacy Concerns

Because Honcho is a hosted/cloud service, using it with real clinical data raises [[concepts/phi-data-handling]] concerns. In the Luria app, any data passed to Honcho should be de-identified or limited to clinician-facing behavioral metadata (e.g., query patterns, UI preferences) rather than patient content. This aligns with the broader [[concepts/local-first-architecture]] and [[concepts/privacy-first-software]] design of the application.

## Key Properties

| Property | Description |
|---|---|
| **Asymmetric observation** | One agent is observed; another queries observations |
| **Passive learning** | Observer does not interrupt the primary interaction |
| **Session-scoped** | Observations are bounded to a workspace/session namespace |
| **Cloud-hosted** | Honcho manages the memory store externally |
| **Preference modeling** | Primary use case is learning user/clinician style |

## Related Concepts

- [[concepts/persistent-memory]] — Long-term memory accumulation across sessions
- [[concepts/multi-agent-orchestration]] — Broader agent coordination patterns
- [[concepts/local-first-architecture]] — Tension with cloud-hosted observation store
- [[concepts/phi-data-handling]] — Privacy constraints on what can be observed
- [[concepts/style-profiles]] — Output of accumulated preference observation
- [[concepts/knowledge-continuity]] — Preserving learned context across interactions
- [[concepts/honcho-ai-peer-observation]] — This concept itself
