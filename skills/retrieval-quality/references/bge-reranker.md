# BGE reranker (BAAI/bge-reranker-v2-m3)

Source: https://huggingface.co/BAAI/bge-reranker-v2-m3 — fetched 2026-08-05.

Default open-weights reranker: cross-encoder, 0.6B params, base model `bge-m3`, strongly multilingual, Apache-2.0, self-hostable. Max sequence length **512 tokens** for the query + passage pair combined.

It outputs a relevance score directly rather than an embedding — it cannot be indexed, only used to rank a candidate set.

## Usage — FlagEmbedding

```python
from FlagEmbedding import FlagReranker

reranker = FlagReranker("BAAI/bge-reranker-v2-m3", use_fp16=True)

pairs = [[query, doc.text] for doc in candidates]
scores = reranker.compute_score(pairs, normalize=True)   # sigmoid -> [0, 1]

ranked = sorted(zip(candidates, scores), key=lambda x: x[1], reverse=True)[:5]
```

## Usage — transformers

```python
from transformers import AutoTokenizer, AutoModelForSequenceClassification

tokenizer = AutoTokenizer.from_pretrained("BAAI/bge-reranker-v2-m3")
model = AutoModelForSequenceClassification.from_pretrained("BAAI/bge-reranker-v2-m3")
model.eval()

inputs = tokenizer(
    [[query, doc.text] for doc in candidates],
    padding=True, truncation=True, max_length=512, return_tensors="pt",
)
scores = model(**inputs).logits.view(-1).float()   # raw logits, unbounded
```

## Scores

Raw logits are unbounded and not comparable across models. Use `normalize=True` (or apply sigmoid manually) to get `[0, 1]`, then calibrate a cut-off threshold on labeled queries. Never hardcode a threshold copied from another model or another corpus.

## Choosing within the family

| Model | Pick when |
|-------|-----------|
| `bge-reranker-v2-m3` | Default. Multilingual, efficient, cheapest to host. |
| `bge-reranker-v2-gemma` | Better multilingual accuracy, larger and slower. |
| `bge-reranker-v2-minicpm-layerwise` | Best accuracy, layer selection to trade quality for speed. |

Start with `v2-m3`. Move up only when Step 6 of the skill shows ranking is still the bottleneck.

## Operational notes

- Truncation is real: a 512-token limit means long chunks get cut. Chunk to comfortably under the limit, or the reranker scores a fragment.
- Batch the pairs; one forward pass per pair is the cost model. Latency scales linearly with the candidate count.
- Pin the model revision and include it in the rerank cache key — a model swap invalidates every cached score.
- Hosted alternatives (Cohere Rerank, Voyage rerank) fit the same slot in the pipeline; the pattern in the skill does not change.
