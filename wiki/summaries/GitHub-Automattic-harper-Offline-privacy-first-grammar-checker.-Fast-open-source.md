---
doc_type: short
full_text: sources/GitHub-Automattic-harper-Offline-privacy-first-grammar-checker.-Fast-open-source.md
---

# Harper: Offline, Privacy-First Grammar Checker

## Overview
Harper is an open-source English grammar checker created as a lightweight, private alternative to tools like Grammarly and LanguageTool. It is designed for speed and minimal resource usage, with all processing happening locally on the user's device.

## Motivation
The author built Harper out of frustration with existing tools:

- **Grammarly**: Expensive, overbearing, context-lacking suggestions, and a significant [[concepts/privacy-first-software]] concern — all text is sent to remote servers and may be used to train large language models.
- **LanguageTool**: Requires gigabytes of RAM and a ~16GB n-gram dataset download; linting even moderate documents takes several seconds.

## Key Features
- **Speed**: Lints documents in milliseconds; long lint times are treated as bugs.
- **Low memory footprint**: Uses less than 1/50th of LanguageTool's memory.
- **Privacy**: Fully offline — no data leaves the user's device. A core principle of [[concepts/privacy-first-software]].
- **Small binary size**: Small enough to run via [[concepts/webassembly]] in the browser at [writewithharper.com](https://writewithharper.com).
- **Extensible core**: Currently English-only, but architected to support additional languages via community contributions.

## Integrations
- Obsidian plugin
- `harper-ls` language server (for supported editors)
- `harper.js` JavaScript library

## Community
- Open-source with contribution guidelines
- Official Discord server
- Logo designed by Lukas Werner

## Related Concepts
- [[concepts/privacy-first-software]] — Central design principle; avoids sending user data to external servers
- [[concepts/webassembly]] — Enables browser-based deployment