---
sources: [summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_PIPELINE.md, summaries/AUTISM_RAG_SYSTEM_DOCUMENTATION.md, summaries/SKILL.md, summaries/README.md, summaries/conversation-export.md, summaries/WORKFLOW_INSTRUCTIONS.md, summaries/TECHNICAL_DOCS.md]
brief: Hybrid search merges semantic vector similarity and keyword matching into a single ranked retrieval pipeline.
---

# Hybrid Search Retrieval

Hybrid search retrieval is a strategy that merges two complementary retrieval signals — **semantic (vector) search** and **keyword (lexical) search** — into a single ranked result set. By combining both approaches, hybrid retrieval captures the strengths of each while mitigating their individual weaknesses.

See also: [[summaries/TECHNICAL_DOCS]], [[concepts/retrieval-augmented-generation]], [[concepts/vector-search]]

---

## Why Hybrid Search?

### Limitations of Pure Semantic Search
- Relies entirely on embedding quality; rare or domain-specific terms may not be encoded well
- Can return conceptually related but literally irrelevant results
- Computationally heavier (requires vector arithmetic over all stored embeddings)

### Limitations of Pure Keyword Search
- Misses synonyms, paraphrases, and conceptual matches
- Sensitive to exact wording; fails on natural language variation
- No notion of semantic relevance, only term overlap

### Hybrid Advantage
Combining both signals produces a ranked list that is simultaneously **semantically aware** and **lexically grounded**, improving precision across diverse query types. In live clinical testing with the [[concepts/pai-knowledge-base]], hybrid retrieval consistently outperformed either approach alone, particularly for complex multi-concept queries like "differential diagnosis depression versus anxiety."

---

## How It Works

### Step 1 — Semantic Search
The query is embedded into a high-dimensional vector (e.g., 768 dimensions via `nomic-embed-text` via Ollama). Cosine similarity is computed between the query vector and all stored document chunk vectors:

```
similarity = (query_vector · doc_vector) / (||query_vector|| × ||doc_vector||)
```

Range: [-1, 1], where 1 = identical, 0 = orthogonal. Vectors must be **normalized before the dot product** — a common implementation bug is skipping normalization, which causes all similarities to appear identical.

### Step 2 — Keyword Search
Stop words are stripped from the query, and the remaining terms are each matched against stored chunks using SQL `LIKE` pattern matching. The number of matched keywords per chunk is counted as a raw text score. This is fast (~50–100 ms) and deterministic.

**Automatic keyword extraction example** (R):
```r
stop_words <- c("the", "a", "an", "and", "or", "in", "on", "for", "of", ...)
keywords <- query |>
  tolower() |>
  str_split("\\s+") |>
  unlist() |>
  str_remove_all("[^a-z]") |>
  setdiff(stop_words)
```

### Step 3 — Score Normalization
Both scores are normalized independently to the [0, 1] range so they can be fairly combined, regardless of their original distributions:

```r
semantic_score_norm = (score - min(score)) / (max(score) - min(score))
text_score_norm     = keyword_matches / max_keyword_matches
```

### Step 4 — Weighted Combination
```
hybrid_score = (semantic_score_norm × semantic_weight) +
               (text_score_norm × text_weight)
```

Weights are tunable. A common default is `semantic_weight = 0.6`, `text_weight = 0.4`, biasing toward semantic understanding while retaining keyword sensitivity.

### Step 5 — Ranked Return
Chunks are sorted by `hybrid_score` descending, and the top-K are returned with all component scores preserved for downstream inspection.

---

## Implementation in R: `pai_hybrid_search()`

The [[concepts/pai-knowledge-base]] implements hybrid search as `pai_hybrid_search()`, operating on a [[concepts/duckdb-as-vector-store]] backend with embeddings stored in [[concepts/parquet-as-knowledge-store]] files. The function signature:

```r
pai_hybrid_search(
  query,
  top_k = 10,
  semantic_weight = 0.5,
  text_weight = 0.5,
  con = NULL,              # DuckDB connection
  return_full_text = FALSE,
  min_text_matches = 1
)
```

The function returns a tibble with columns for `doc_id`, `chunk_id`, `semantic_score`, `text_score`, `hybrid_score`, `text_preview`, and optionally `full_text`. The [[concepts/duckdb-as-vector-store]] backend stores all 2,546 chunk embeddings (768-dimensional JSON arrays) and supports the SQL keyword search without loading data into R memory.

---

## Performance Profile

In the PAI RAG system (see [[summaries/conversation-export]]):

| Search Mode | Latency |
|---|---|
| Keyword only | 50–100 ms |
| Semantic only | 500–800 ms |
| Hybrid | 600–900 ms |

The overhead of hybrid over pure semantic is modest (~100 ms), while the accuracy benefit is significant:

| Clinical Query Type | Keyword Matches | Top Hybrid Score |
|---|---|---|
| Depression symptoms (DEP/DEP-C) | 285 chunks | 0.500 |
| Validity / malingering detection | 1,965 chunks | 0.700 |
| Anxiety + physiological arousal | 400 chunks | 0.500 |
| Complex comorbid case | 2,007 chunks | 0.650 |

Semantic scores for good matches ranged from **0.65–0.74**, confirming that normalized cosine similarity provides a strong signal for domain-specific retrieval when embeddings are computed correctly.

---

## Tuning the Hybrid Balance

- **High semantic weight (e.g., 0.8/0.2)**: Better for conceptual or exploratory queries (e.g., "What constructs relate to emotional dysregulation?", "What indicates poor treatment prognosis?")
- **High keyword weight (e.g., 0.3/0.7)**: Better for exact-term lookups — scale abbreviations like `BOR`, `DEP-C`, `ICN`, or `NIM`
- **Balanced (0.5/0.5)**: Reasonable default for unknown query distributions; recommended for general clinical use
- **Semantic-only (1.0/0.0)**: Appropriate when the query contains no known terminology (e.g., colloquial descriptions of symptoms)

A key empirical finding from PAI clinical testing: **the same top chunks typically appear across weight configurations** — weights mainly affect ranking order, not which documents are discovered. This means balanced weights are safe as a default.

---

## Common Implementation Pitfalls

| Bug | Symptom | Fix |
|---|---|---|
| Missing vector normalization | All cosine similarities are identical | Divide each vector by its L2 norm before dot product |
| Wrong matrix indexing in R | 768-D embeddings stored as 1-D scalars | Use `matrix[i, ]` (row) not `matrixi` (list element) |
| JSON serialization of list element | Only first value serialized | Extract numeric vector with `as.numeric()` before `toJSON()` |
| Stopword list too aggressive | Query terms like `PAI`, `NIM` stripped | Curate stop word list for domain vocabulary |

---

## Integration with the Full RAG Pipeline

In a complete [[concepts/retrieval-augmented-generation]] pipeline, hybrid search feeds its top-K results into a context formatter that:

1. Concatenates chunk text with source labels (e.g., `--- Source 1 --- (Relevance: 0.70)`)
2. Embeds the formatted context into a structured LLM prompt with a clinical system role
3. Routes to the appropriate LLM provider via [[concepts/llm-provider-abstraction]]

For the [[concepts/neuropsychological-assessment-pipeline]], this enables queries such as:
- *"When should I invalidate a PAI profile due to inconsistent responding?"*
- *"What does an elevated BOR scale tell me about the patient?"*
- *"Patient has elevated DEP and ANX with normal BOR — what's the clinical picture?"*

---

## Optimization Paths

As document collections scale, the keyword search component can be accelerated by replacing SQL `LIKE` with a **precomputed inverted index** stored in a separate table and joined at query time. For very large corpora (250K+ chunks), the semantic search component benefits from **approximate nearest neighbor (ANN)** methods such as FAISS or Annoy, trading marginal accuracy for 10–100× speed improvements.

For clinical applications where privacy is paramount, all components described here run entirely locally: [[concepts/local-llm-inference]] via Ollama, local embedding models, and [[concepts/duckdb-as-vector-store]] with no external API calls required.

---

## Related Concepts

- [[concepts/retrieval-augmented-generation]] — Hybrid search is the retrieval layer in RAG pipelines
- [[concepts/vector-search]] — The semantic half of hybrid retrieval
- [[concepts/duckdb-as-vector-store]] — Storage backend for embeddings and chunks
- [[concepts/parquet-as-knowledge-store]] — Columnar format for compressed embedding storage
- [[concepts/pai-knowledge-base]] — Domain knowledge base where hybrid search is applied
- [[concepts/llm-provider-abstraction]] — Consumer of retrieved context in the full pipeline
- [[concepts/neuropsychological-assessment-pipeline]] — Clinical application context
- [[concepts/local-llm-inference]] — Enables fully private, on-device retrieval pipelines
- [[concepts/pai-assessment]] — The instrument whose documentation is indexed and retrieved

See also: [[summaries/WORKFLOW_INSTRUCTIONS]], [[summaries/conversation-export]]

See also: [[summaries/README]]

See also: [[summaries/SKILL]]

See also: [[summaries/AUTISM_RAG_SYSTEM_DOCUMENTATION]]

See also: [[summaries/NEUROPSYCHOLOGICAL_REPORT_RAG_PIPELINE]]