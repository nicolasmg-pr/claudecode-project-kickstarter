---
name: retrieval-quality
description: Improve RAG and search result quality with retrieve-then-rerank, hybrid retrieval, and measured evaluation. Use when building or debugging RAG, vector search, or semantic search, or when retrieved context is off-topic, incomplete, or the model answers from the wrong document.
argument-hint: "[the retrieval step to improve]"
---

# Retrieval Quality

Vector search optimizes for recall, not precision. A bi-encoder compresses the query and the document into vectors independently, so it never sees them together — near-misses rank as high as exact answers. **Reranking is the main precision lever.** Apply it before rewriting chunking, swapping the embedding model, or enlarging the context window.

## Step 1 — Diagnose before changing anything

Take 20 real queries. For each, log the top-10 retrieved chunks and mark whether the correct chunk is present and at which rank.

- Correct chunk **absent** from top-50 → recall problem. Fix retrieval (chunking, hybrid search, query rewriting). Reranking cannot recover what was never retrieved.
- Correct chunk **present but ranked low** → precision problem. Rerank.

Skipping this step means guessing which half of the pipeline is broken.

## Step 2 — Retrieve wide, rerank narrow

```
query → retrieve top 50-100 (bi-encoder / BM25 / hybrid) → cross-encoder rerank → top 5-10 → LLM
```

- The cross-encoder scores query and passage **jointly**, so it catches negation, entity mismatch, and specificity that cosine similarity misses.
- Retrieve wide enough that recall is high (50-100), pass few enough that the LLM stays focused (5-10). Feeding 50 chunks to the model is not a substitute — it costs more and degrades answer quality.
- Cross-encoders cost one forward pass per candidate. That is why they run on 50 candidates, never on the whole index.

Model and code: [references/bge-reranker.md](bge-reranker.md).

## Step 3 — Hybrid retrieval for the candidate set

Dense retrieval alone misses exact identifiers, error codes, product SKUs, and rare terms. Run BM25/keyword alongside vector search, merge with Reciprocal Rank Fusion, then rerank the merged set. The reranker resolves the ordering, so the merge only needs to be generous, not perfect.

## Step 4 — Cut the score threshold

The reranker's score is usable as an absolute relevance signal (normalize to `[0,1]` first). Drop candidates below the threshold instead of always returning top-k — returning three good chunks beats padding to ten with noise. Calibrate the threshold on the labeled set from Step 1.

## Step 5 — Budget latency and cost

- Reranking 50 candidates adds roughly 50-200 ms on GPU, seconds on CPU. Measure on the real corpus, not a toy example.
- Cache rerank scores keyed on `(query_hash, doc_id, model_version)` — repeated and identical-input queries are exactly the case the caching-strategy skill covers.
- Too slow → reduce the candidate set before reducing model quality. Going 100 → 50 candidates usually costs little accuracy.

## Step 6 — Prove it moved

Re-run the Step 1 query set and report the change in hit rate @5 and MRR. Both flat → the bottleneck is recall or chunking, not ranking. Revert the reranker rather than keeping unmeasured complexity.
