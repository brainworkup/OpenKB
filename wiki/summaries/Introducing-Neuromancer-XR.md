---
doc_type: short
full_text: sources/Introducing-Neuromancer-XR.md
---

# Introducing Neuromancer XR

Meet Neuromancer XR--our custom reasoning model that achieves state-of-the-art memory by extracting & scaffolding logical conclusions from conversations.

Memory is a foundational pillar of social cognition. As a key component of [Honcho](https://honcho.dev/), we approach it as a combined reasoning and retrieval problem.

Neuromancer XR: Training a Logical Reasoning Specialist for Memory

To implement this vision, we need a model that can reliably extract and categorize conclusions from conversations. Our initial focus for the memory task, given its focus on factual recall, is on the first two certainty levels: explicit and deductive knowledge--that is, conclusions we know to be true given what users (or agents) state in their messages.

We generated a proprietary dataset of approximately 10,000 manually curated instances of conclusion derivation, creating memory-reasoning traces from conversational data. Each instance shows how to process a conversation turn and derive the relevant conclusions at appropriate certainty levels. We then fine-tuned Qwen3-8B on these traces.

The resulting model is Neuromancer XR (for eXplicit Reasoning), a model specialized in deriving explicit and deductive conclusions from conversational data. It is currently in production powering the latest release of [Honcho](https://www.honcho.dev/).

Evaluation

Although the Honcho workflow allows us to answer any arbitrary question about a peer, from the purely factual to the predictive, it's important for us to be able to benchmark its raw memory abilities--how accurately it can recall factual information shared by a user in a conversation.

We’re interested, specifically, in evaluating whether using Neuromancer XR for the conclusion derivation step would result in better memory performance, compared to (1) the base model used for the fine-tune (Qwen3-8B), and(2) a reasonable frontier baseline, for which we picked Claude 4 Sonnet for its aptitude at this task.

To this end, we tested Honcho on the [LoCoMo](https://arxiv.org/abs/2402.17753) memory benchmark. While we're aware that it has a number of shortcomings, including outdated rule-based scoring, insufficient length, and ambiguous or ill-posed questions, we have found it to be a reasonable benchmark for research and development when paired with (1) a carefully crafted LLM-as-judge prompt, which we include in Appendix A, and (2) rigorous manual inspection of evaluation traces.

Directions for future work

We're training a model for the remaining two levels of logical certainty outlined above in our framework: inductive and abductive. The next model in the Neuromancer series, Neuromancer MR (for meta-reasoning), will be in charge of this.

This model will reason about reasoning, focusing on the predictive side of the certainty spectrum. It will allow us to derive likely explanations and probable hypotheses for broad patterns of user or agent behavior at the moment of ingestion, bolstering the density and utility of peer representations.

## Related Concepts
- [[concepts/concept-slug]]
- [[concepts/concept-slug]]
- [[concepts/concept-slug]]
- [[concepts/concepts-attention]]
- [[concepts/explorations-social-cognition]]
- [[concepts/academic-accommodations-higher-education]]
