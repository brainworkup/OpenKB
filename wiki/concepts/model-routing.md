---
sources: [summaries/MIGRATION_GUIDE.md]
brief: Route tasks to specialized models by semantic role instead of a single all-purpose LLM.
---

# Model Routing by Semantic Role
Rather than using one large model for every task (embedding, generation, fallback), route each call by its *semantic role* to the best suited tool. This is what powers the current codebase:

## The mechanism in `core`
- **Embeddings**: Routed through specialized embedding models (`Qwen3-Embedding-8B-4bit`) with a vector store and FAISS fallback for high-speed semantic retrieval.\
- **Generation**: Text generation routes to the strongest reasoning model available, but includes *generation fallbacks* — if one server is down or slow, automatically route to an alternative (e.g., Qwen3.5 via OMLX).
- **Data Paths**: Each step has a fallback path so the pipeline remains reliable even when specific providers are unreachable.\n\
## Why it matters\
Using a single LLM for everything is brittle and expensive. Semantic routing gives you:\
- **Speed/Cost**: Smaller embedding models handle retrieval at scale.\
- **Quality**: Large reasoning models only get called for generation, not the whole pipeline.\
- **Reliability**: Fallbacks prevent hard failures when APIs change or servers drop.\n\
## Where to see it in code\
- `core/vector_ops.py`: contains both fast and fallback paths (`embed_with_fallback`, `generate_with_fallback`).\
- `core/database.py`: routes queries between SQLite, FAISS (if present), and in-memory cosine similarity.
- The generation route defaults to the OMLX endpoint but can be overridden per call via `--omlx-*` CLI flags during report writing.\n\
**Related**: [[concepts/local-inference-reliability]], [[concepts/llm-provider-abstraction]], [[summaries/MIGRATION_GUIDE]]