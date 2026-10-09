# kb on DRAGON (hist, version 1.15.0)

Third-party result for [kb](https://github.com/alterfo/kb), a self-contained graphRAG knowledge base written in Go (hybrid dense + BM25 retrieval, entity graph, Graph-of-Thoughts answer synthesis; one SQLite file for vectors, graph and metadata).

This is a self-run submission scored with the official `rag_bench.evaluator.evaluate_rag_results`, not a leaderboard entry produced by the hosted backend.

## Setup

- Dataset: `ai-forever/hist-rag-bench-*`, revision `1.15.0`, 542 texts, 600 questions (150 per type).
- Chat / graph extraction: `qwen3.8` (architecture qwen35, 27.3B, Q4_K_M) served by Ollama, thinking disabled. The index was rebuilt from scratch with this model.
- Embeddings: `qwen3-embedding:0.6b`.
- Index: all 542 texts indexed with entity/relation extraction and community summaries.
- Answering: retrieval and Graph-of-Thoughts synthesis, followed by one extra chat call that reduces the answer to its bare value (name, number, date or short list), so that Exact Match, Substring Match and ROUGE are meaningful. `found_ids` are the retrieved public text ids in ranked order.
- Local hardware only, no external API calls.

## Results

| Metric | Overall | cond | mh | set | simple |
|---|---|---|---|---|---|
| Hit Rate | 0.9153 | 0.9167 | 0.8911 | 0.9333 | 0.9200 |
| MRR | 0.6279 | 0.5849 | 0.6147 | 0.6944 | 0.6175 |
| ROUGE-1 | 0.6660 | 0.8315 | 0.6305 | 0.5754 | 0.6265 |
| ROUGE-2 | 0.4496 | 0.6117 | 0.3559 | 0.3547 | 0.4761 |
| ROUGE-L | 0.6289 | 0.8255 | 0.6296 | 0.4338 | 0.6265 |
| Exact Match | 0.3550 | 0.7067 | 0.3667 | 0.0000 | 0.3467 |
| Substring Match | 0.4300 | 0.7400 | 0.4600 | 0.0467 | 0.4733 |

## Files

- `submission.json`: `{public_id: {found_ids, model_answer}}`, exactly the structure `evaluate_rag_results` expects.
- `official-report.json`: `average_metrics` returned by the evaluator, same numbers as above.

## Reproduce the scoring

```python
import json
from datasets import load_dataset
from rag_bench import evaluator

version = "1.15.0"
results = {k: {"found_ids": [int(x) for x in v["found_ids"]], "model_answer": v["model_answer"]}
           for k, v in json.load(open("submission.json")).items()}
qa = load_dataset("ai-forever/hist-rag-bench-private-qa", revision=version)
texts = load_dataset("ai-forever/hist-rag-bench-private-texts", revision=version)
mapping = {t["public_id"]: t["id"] for t in texts["train"]}
print(evaluator.evaluate_rag_results(results, qa, mapping).average_metrics)
```

## Notes

- The `set` type scores near zero on Exact Match because list answers are compared as a whole string, so any difference from the gold list fails the match.
- The short-answer reduction step is part of the local kb build used for this run and has not been published in the kb repository yet. The submission file and the scoring above can be verified independently of it.
