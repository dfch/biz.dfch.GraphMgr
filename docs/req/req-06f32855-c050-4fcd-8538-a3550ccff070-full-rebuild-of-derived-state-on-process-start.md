---
classification: null
created: '2026-10-13T04:14:20.620+02:00'
id: 06f32855-c050-4fcd-8538-a3550ccff070
status: draft
type: req
updated: '2026-10-13T04:14:20.620+02:00'
version: 1.0.0
---

# Full Rebuild of Derived State on Process Start

WHEN the process starts, GraphMgr shall rebuild the change-detection manifest, the relational query index, and the vector embeddings from the text files from scratch; GraphMgr shall not persist derived state across restarts.

## Description

This is the single consistency invariant for all derived state: process start means a guaranteed-correct full resync. The trade-off is a slower cold start as the corpus grows (embedding recomputation is the first genuinely slow part at scale); a persistent embedding cache keyed by content hash was considered and explicitly rejected to keep the invariant uniform.

## Characteristics

1. Reliability
2. Flexibility

## Level

MUST

## Priority

20

## Tags

- change-detection
- cold-start

## Source

First design discussion 2026-10-10/11 (first-conversation.md sections 4.3, 5.7, and 6); DEC 4a3b078a-852c-4a4e-87b5-816e17dd0a44.

## Related Artifacts

### Decisions

- DEC 4a3b078a-852c-4a4e-87b5-816e17dd0a44: Derived State and Query Layer: Full Rebuild on Startup, SQLite with sqlite-vec
- DEC 3cccd4cb-f488-4a6f-8005-f38d2b5e92ef: Change Detection: Git-Independent Filesystem Manifest with Hybrid Watch and Poll

### Risks

- RSK 8401d706-a3dd-4205-9675-f73daec6d144: Cold-Start Degradation from Full Rebuild and Full Re-Embedding
