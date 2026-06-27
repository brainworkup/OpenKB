---
sources: [summaries/mlx_embeddings.md]
brief: A multi-vector retrieval paradigm where token-level embeddings are matched via MaxSim scoring.
---

# Late Interaction Retrieval

Late interaction retrieval is a neural retrieval paradigm in which queries and documents are each represented as **sets of token-level embeddings** rather than a single dense vector. Similarity is computed by aggregating fine-grained token-to-token matches at query time, preserving more semantic detail than single-vector bi-encoders while remaining far more efficient than full cross-attention re-rankers.

## How It Works

### Single-Vector vs. Late Interaction

| Approach | Query Rep | Document Rep | Match |  
|---|---|---|---|  
| Bi-encoder (dense) | 1 vector | 1 vector | dot product |  
| Cross-encoder | — | — | full attention over concatenated pair |  
| Late interaction | N token vectors | M token vectors | MaxSim aggregation |

### MaxSim Scoring

The canonical scoring function is **MaxSim**:

```
Score(Q, D) = Σ_i  max_j ( q_i · d_j )
```

For each query token embedding `q_i`, find the document token embedding `d_j` that is most similar, then sum these maximum similarities across all query tokens. This allows every query token to "find" its best matching document token independently.

In MLX (from [[summaries/mlx_embeddings]]):

```python
def maxsim(query_embeds, image_embeds):
    sims = query_embeds @ image_embeds.T
    return mx.sum(mx.max(sims, axis=1))
```

## Key Models

### ColPali / ColQwen

ColPali and ColQwen (e.g., `colqwen2.5-v0.2`) are **multimodal late interaction models** that apply this paradigm to image retrieval:

- **Query side:** Text tokens are embedded via a language model backbone; a projection layer maps hidden states to the interaction space.
- **Document side:** Images are encoded as visual token sequences (via a vision encoder), producing one embedding per image patch or visual token.
- **Matching:** MaxSim scores each query token against all visual tokens of each candidate image.

This enables fine-grained image retrieval without requiring images to be summarised into a single vector, preserving spatial and semantic structure.

## Pipeline (MLX Implementation)

The pipeline as shown in [[summaries/mlx_embeddings]] involves several manual steps:

1. **Tokenise query** — prepend `"Query: "` prefix; pad with extra pad tokens for regularisation.
2. **Embed query tokens** — `model.get_input_embeddings_batch(input_ids)`
3. **Compute RoPE position IDs** — `model.vlm.language_model.get_rope_index(...)`
4. **Run language model** — `model.vlm.language_model.model(...)` to get hidden states.
5. **Project & normalise** — `normalize_embeddings(model.embedding_proj_layer(hidden))`
6. **Mask padding** — multiply embeddings by attention mask to zero out pad tokens.
7. **Remove padding rows** — retain only non-padded token embeddings.

For images, `pixel_values` and `image_grid_thw` are additionally passed to `get_input_embeddings_batch` before the same pipeline.

## Advantages

- **Expressiveness:** Captures token-level alignment; effective for complex queries.
- **Efficiency:** Documents can be pre-indexed as token embedding matrices offline; only MaxSim is computed at retrieval time.
- **Multimodal generalisation:** Visual tokens and text tokens inhabit the same interaction space, enabling cross-modal retrieval.

## Relationship to Other Concepts

- [[concepts/multimodal-embeddings]] — late interaction extends dense multimodal embeddings to multi-vector representations.
- [[concepts/retrieval-augmented-generation]] — late interaction models serve as powerful retrievers feeding context into generation pipelines.
- [[concepts/mlx-framework]] — the MLX runtime enables on-device late interaction inference on Apple Silicon.
- [[concepts/model-quantization]] — quantising ColPali/ColQwen models reduces memory for storing large token embedding indices.
- [[concepts/local-llm-inference]] — running late interaction retrieval locally preserves privacy and avoids API latency.

## References

- [[summaries/mlx_embeddings]] — ColQwen2.5 usage example including `maxsim`, `embed_query`, and `embed_image` implementations.
