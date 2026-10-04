# kb on DRAGON (hist, version 1.15.0)

Third-party result for [kb](https://github.com/alterfo/kb), a self-contained graphRAG knowledge base written in Go (hybrid dense + BM25 retrieval, entity graph, Graph-of-Thoughts answer synthesis; one SQLite file for vectors, graph and metadata).

This is a self-run submission scored with the official `rag_bench.evaluator.evaluate_rag_results`, not a leaderboard entry produced by the hosted backend.

## Setup

- Dataset: `ai-forever/hist-rag-bench-*`, revision `1.15.0`, 542 texts, 600 questions (150 per type).
- Chat / graph extraction: `qwen3.8:latest` (27B, Q4_K_M) served by Ollama, thinking disabled.
- Embeddings: `qwen3-embedding:0.6b`.
- Index: all 542 texts indexed with entity/relation extraction and community summaries.
- Answering: retrieval and Graph-of-Thoughts synthesis, followed by one extra chat call that reduces the answer to its bare value (name, number, date or short list), so that Exact Match, Substring Match and ROUGE are meaningful. `found_ids` are the retrieved public text ids in ranked order.
- Local hardware only, no external API calls.

## Results

| Metric | Overall | cond | mh | set | simple |
|---|---|---|---|---|---|
| Hit Rate | 0.8975 | 0.9167 | 0.8878 | 0.9056 | 0.8800 |
| MRR | 0.6035 | 0.6122 | 0.6031 | 0.6037 | 0.5950 |
| ROUGE-1 | 0.6441 | 0.7630 | 0.6391 | 0.5605 | 0.6137 |
| ROUGE-2 | 0.4319 | 0.5598 | 0.3838 | 0.3259 | 0.4583 |
| ROUGE-L | 0.6048 | 0.7597 | 0.6357 | 0.4131 | 0.6107 |
| Exact Match | 0.3017 | 0.5667 | 0.4067 | 0.0000 | 0.2333 |
| Substring Match | 0.3500 | 0.5933 | 0.4333 | 0.0067 | 0.3667 |

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
