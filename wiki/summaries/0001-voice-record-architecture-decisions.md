---
doc_type: short
full_text: sources/0001-voice-record-architecture-decisions.md
---

# Summary: 0001 — Record Architecture Decisions

**Date:** 2025-01-20
**Status:** Accepted

## Overview

This is the foundational [[concepts/architecture-decision-records]] (ADR) for the voice project. It establishes the practice of capturing significant architectural decisions as lightweight, immutable documents so that rationale is never lost between sessions or contributors.

## Problem Being Solved

The voice project integrates several complex subsystems:

- [[concepts/typst-typesetting]] — document typesetting templates
- [[concepts/quarto]] — document rendering pipeline
- Retrieval-augmented generation-based style agents (see [[concepts/retrieval-augmented-generation]])
- [[concepts/brand-theming]] — brand and visual theming

Each subsystem carries non-obvious design trade-offs. Without explicit records, the reasoning behind decisions evaporates across sessions and as contributors change.

## Decision

Adopt the **Michael Nygard ADR template** with the following properties:

- Stored in `docs/adr/`
- Numbered sequentially
- Follow the structure: **Status → Context → Decision → Consequences**
- Immutable after acceptance — superseding decisions create a new ADR referencing the old one

## Key Consequences

| Outcome | Detail |
|---|---|
| Discoverability | Every significant choice has a findable rationale |
| Onboarding | New contributors or future sessions can reconstruct intent without reading all source code |
| Immutability | Accepted ADRs are never edited; change requires a new ADR |

## Key Concepts

- [[concepts/architecture-decision-records]] — the practice and format being adopted
- [[concepts/knowledge-continuity]] — the underlying motivation: preserving intent across time and contributors
- [[concepts/documentation-as-code]] — the broader practice this ADR pattern supports

{"brief": "Establishes ADR practice for the voice project to preserve architectural rationale across sessions.", "content": "already returned above"}

## Related Concepts
- [[concepts/plain-text-documentation]]
