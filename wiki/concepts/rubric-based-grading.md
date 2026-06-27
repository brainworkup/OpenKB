---
sources: [summaries/Introducing-FrontierCode.md]
brief: Grading AI outputs using structured quality criteria and LLM judges beyond binary pass/fail correctness.
---

# Rubric-Based Grading for AI Outputs

Rubric-based grading is an evaluation methodology that assesses AI-generated outputs against a structured set of quality criteria, moving beyond binary correctness checks toward nuanced, multi-dimensional scoring. It is especially relevant for evaluating [[concepts/ai-code-generation]] outputs where correctness is necessary but insufficient.

## Why Binary Testing Falls Short

Traditional unit tests produce a binary verdict: a solution either passes or fails. This works well for deterministic, narrowly-scoped problems but breaks down when:

- **Multiple valid solutions exist** (different function names, equivalent algorithms, alternate error messages)
- **Quality dimensions matter** beyond correctness (readability, idiomatic style, architectural soundness)
- **Subjective expert judgment** is required to distinguish a good PR from a merely functional one

Rubric-based grading resolves these limitations by introducing a *spectrum* of correctness: two solutions for the same task can both be functionally correct yet score differently on style, scope, or maintainability dimensions.

## Structure of a Rubric

A well-designed rubric decomposes evaluation into individual **criteria**, each of which is assigned:

- A **grading method** (unit test, LLM prompt, shell command, scope check)
- A **weight** reflecting its importance relative to other criteria
- A **blocker vs. non-blocker** classification

### Blockers vs. Non-Blockers

| Type | Description | Effect on Score |
|---|---|---|
| **Blocker** | Hard stop — a maintainer would reject the PR | Failure → score = 0 |
| **Non-blocker** | Quality signal — desirable but not mandatory | Partial credit |

In [[summaries/Introducing-FrontierCode]], blockers include correctness checks, performance constraints, and scope restrictions. Non-blockers cover code style, type safety, and readability.

## Grading Methods Used in Practice

The FrontierCode benchmark employs a novel ensemble of grading techniques:

| Method | Used For | How It Works |
|---|---|---|
| Classical unit tests | Behavioral correctness | Injects test files, runs them, checks exit status |
| Reverse-classical | Test quality | Runs agent tests against broken base — must fail |
| Adaptive classical (`mutagent`) | Open-ended correctness | LLM patches tests to align with agent's implementation |
| Scope checks | PR discipline | File allowlists, size limits, semantic locality |
| LLM prompt grading | Code quality / style | LLM reviews diff against a natural-language rubric |
| Shell command | Mechanical cleanliness | Checks lint, build, regression safety via exit code |

## The Subjective Grading Problem

Rubric design is inherently subjective and requires domain expertise. Key design challenges include:

- **Coverage**: Rubrics must be complete enough that models cannot exploit gaps.
- **Weight calibration**: Criteria must be weighted to reflect real-world maintainer priorities.
- **Blocker classification**: Deciding which criteria are hard stops vs. quality signals requires expert judgment.
- **Resolution**: A well-calibrated rubric should separate solutions across the full 0–100% score range.

To validate rubric quality, FrontierCode's process requires authors to write **four distinct solutions** targeting different score levels, from 0% to 100%.

## Quality Control for Rubrics

Hardening rubric-based criteria requires a different approach than hardening unit tests:

1. **Hack reports**: Authors attempt to game the rubric with deliberately incomplete solutions (false positive detection) and write valid alternative solutions that differ from the canonical one (false negative detection).
2. **Adversarial testing**: Tools like Devin are used to find novel rubric exploits.
3. **Multi-stage review**: Pod leads, then senior researchers, review rubric design across multiple iteration rounds.
4. **Calibration solutions**: Four distinct solutions are required to verify rubric resolution.

This process produces an **81% lower false positive rate** compared to SWE-Bench Pro, according to [[summaries/Introducing-FrontierCode]].

## When to Prefer LLM Grading vs. Classical Tests

- **Prefer classical tests** for deterministic, verifiable behaviors (function output, exit codes, test passage).
- **Prefer LLM grading** for soft qualities: idiomatic code, readability, adherence to architectural patterns, codebase conventions.
- **Use adaptive classical grading** when the task is open-ended but a deterministic check is still desirable — bridging the gap between rigidity and flexibility.

## Relationship to Other Concepts

Rubric-based grading is a core technique in [[concepts/llm-evaluation]] more broadly. It shares principles with:

- [[concepts/ai-code-generation]] — the domain where rubric grading is applied in FrontierCode
- [[concepts/multi-agent-orchestration]] — rubrics can evaluate agent behavior at the system level
- [[concepts/scaling-laws]] — rubric scores can reveal whether model capability improvements translate to real-world quality
- [[concepts/report-review-qa]] — rubric-style review criteria appear in clinical report quality assurance contexts as well

## Key Insight

Rubric-based grading is not simply "LLM-as-judge" scoring. At its best, it is a **structured expert knowledge capture** process: domain experts encode years of judgment about what makes an output high-quality into explicit, weighted, testable criteria. The rubric becomes a durable artifact of human expertise that can be applied consistently and at scale.
