---
doc_type: short
full_text: sources/Introducing-FrontierCode.md
---

# Introducing FrontierCode

## Overview

FrontierCode is a code benchmarking framework introduced by Cognition to evaluate whether AI-generated code meets the standards of high-quality, production-ready codebases. Unlike previous benchmarks that test only functional correctness, FrontierCode asks: *would a maintainer actually merge this PR?*

## Motivation

Existing benchmarks like SWE-Bench Verified and SWE-Bench Pro were designed for less capable models and have several critical shortcomings:

- They test only **functional correctness**, not code quality.
- They suffer from high **misclassification error rates** (false positives and false negatives).
- They provide **overly detailed prompts**, giving more guidance than a real contributor would receive.
- They lack **linguistic and repository diversity**.

METR experiments showed that high-scoring models on existing benchmarks often produce patches that real maintainers would reject. FrontierCode achieves **81% fewer misclassification errors** than SWE-Bench Pro.

## Benchmark Structure

FrontierCode consists of 150 tasks organized in three nested subsets by difficulty:

| Subset | Tasks |
|---|---|
| Extended | 150 (full set) |
| Main | 100 hardest |
| Diamond | 50 hardest |

### Metrics
- **Pass rate**: A solution passes if it clears all *blocker* criteria.
- **Score**: A weighted aggregate of all rubric items (0 if any blocker fails).

Each model is run 5 times at every available reasoning effort; the best effort level is reported.

## Results

FrontierCode Diamond remains far from saturated:

| Model | Diamond Score | Main Score | Extended Score |
|---|---|---|---|
| Claude Opus 4.8 | 13.4% | 34.3% | 51.8% |
| GPT-5.5 | 6.3% | — | — |
| Gemini 3.1 Pro | 4.7% | — | — |
| Kimi K2.6 (best open-source) | 3.8% | 16% | 37% |

Noteworthy: GPT-5.5 uses up to 4× fewer tokens than Opus 4.8, offering a better cost-intelligence tradeoff.

## Evaluation Axes

FrontierCode evaluates code along six dimensions:

1. **Behavioral correctness** — Does the patch solve the problem?
2. **Regression safety** — Does it break existing functionality?
3. **Mechanical cleanliness** — Does it pass build, lint, and style checks?
4. **Test correctness** — Do the agent's tests capture the desired behavior?
5. **Scope** — Does the patch touch only what it needs to?
6. **Code quality** — Does it conform to codebase conventions and design patterns?

Criteria are classified as **blockers** (hard stops) or **non-blockers** (quality signals).

## Novel Grading Methods

### Reverse-Classical
Agent-written tests are run against the *original broken codebase*. They must fail, proving the agent understood the problem well enough to write a meaningful test.

### Code Scope
Automated checks enforce PR restraint via:
- `files`: allowlists/denylists for modified files
- `size`: limits on changed lines, net growth, or total files
- `semantic`: LLM-based checks for locality of changes within a file

### Adaptive Classical Grading (`mutagent`)
An LLM surgically patches the test environment or application code to align with the agent's implementation details (e.g., different function names or error strings), enabling rigorous deterministic tests on open-ended solutions.

## Construction Process

- **20+ world-class open-source maintainers** from 36 flagship repos (e.g., Celery, Budibase, uppy, Mattermost)
- **40+ hours per task**, with multiple iteration rounds
- Tasks hand-selected from multi-PR chains and freeform requests
- Prompts are **deliberately concise** (~⅓ the length of SWE-Bench Pro's)
- Difficulty scaled via **quality rubrics**, not patch size

## Quality Control Pipeline

1. **Design** — Prefer classical tests for deterministic checks; LLM grading for soft qualities.
2. **Hack Report** — Task authors attempt to game the rubric (adversarial testing); Devin also used to find novel exploits.
3. **Rubric Calibration** — Authors write four solutions targeting a range from 0–100%.
4. **Review** — Pod lead review, then final Cognition researcher review.
5. **Re-Review** — Tasks cycle through multiple iterations; most tasks revised before passing.

## Key Differentiators vs. Prior Benchmarks

| Feature | FrontierCode | SWE-Bench Pro |
|---|---|---|
| Tests quality | ✅ | ❌ |
| Maintainer-crafted tasks | ✅ | ❌ |
| Misclassification rate | 81% lower | Baseline |
| Prompt length | Concise (humanlike) | Verbose |
| Language diversity | 3× SWE-Bench Pro | Lower |
| Novel grading methods | ✅ | ❌ |

## Related Concepts
- [[concepts/scaling-laws]]
- [[concepts/retrieval-augmented-generation]]
- [[concepts/multi-agent-orchestration]]

- [[concepts/ai-code-generation]]
- [[concepts/llm-evaluation]]
- [[concepts/rubric-based-grading]]