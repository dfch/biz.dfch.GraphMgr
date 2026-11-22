---
classification: null
created: '2026-10-13T04:03:16.781+02:00'
id: 8401d706-a3dd-4205-9675-f73daec6d144
status: open
type: rsk
updated: '2026-10-13T04:03:16.781+02:00'
version: 1.0.0
---

# Cold-Start Degradation from Full Rebuild and Full Re-Embedding

## Cause

The derived-state decision mandates rebuilding the change-detection manifest, the SQLite query index, and all vector embeddings from scratch on every process start. Embedding generation is CPU-bound and non-trivial per item, unlike the sub-millisecond stat/hash checks used for the rest of the sync.

## Trigger

The corpus grows (for example 100k+ entities) and the process restarts for a deploy, update, or crash.

## Consequence

Startup time grows with the corpus size; embedding recomputation becomes the dominant cost. The service is slow to serve correct query results until the rebuild completes.

## Scope

- Query layer (relational index and vector index)
- Operations (deploys, uptime, maintenance windows)

## Initial Assessment

### Probability 3

### Impact 3

## Strategy

reduce

## Mitigation

Parallelize the full-tree stat/hash walk per shard (shards are independent directories with no shared state). Benchmark cold start at target scale during implementation. Keep the single "persist nothing, rebuild everything" consistency invariant; revisit only if measurements force it, in which case a persistent embedding cache keyed by content hash would be an explicit, documented exception.

## Residual Assessment

### Probability 2

### Impact 3

## Owner

Development lead (to be assigned).

## Tags

- performance
- cold-start
- scale

## Source

First design discussion, 2026-10-10/11 (first-conversation.md sections 4.3 and 5.7).

## More Information

See DEC 4a3b078a-852c-4a4e-87b5-816e17dd0a44: Derived State and Query Layer: Full Rebuild on Startup, SQLite with sqlite-vec. The user explicitly chose full recomputation over a persistent embedding cache, accepting the slower cold start.
