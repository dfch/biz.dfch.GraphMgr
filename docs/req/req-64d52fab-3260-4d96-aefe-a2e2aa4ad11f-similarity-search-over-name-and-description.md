---
classification: null
created: '2026-10-13T04:14:49.114+02:00'
id: 64d52fab-3260-4d96-aefe-a2e2aa4ad11f
status: draft
type: req
updated: '2026-10-13T04:14:49.114+02:00'
version: 1.0.0
---

# Similarity Search over Name and Description

GraphMgr shall support similarity search of nodes and edges based on their name and description, using locally computed embeddings stored via the sqlite-vec extension in the same SQLite database; GraphMgr shall make no external API calls for embedding computation.

## Description

Each entity's name and description text is embedded with a small local model (fastembed / bge-small class, as precedent exists in specmgr's own find_related and find_similar_text tools). Embeddings are another derived, rebuildable index: they piggyback on the same change-detection pipeline (recompute and upsert when the index file's content changes) and are fully recomputed on every startup. Query results can be joined with normal SQL or graph filters (for example: similar nodes, but only of type X). A dedicated embedded ANN library is the fallback if sqlite-vec proves insufficient; an external vector-DB service was rejected as an inconsistent operational footprint.

## Characteristics

1. Functional Suitability
2. Performance Efficiency

## Level

MUST

## Priority

30

## Tags

- query
- embeddings
- similarity

## Source

First design discussion 2026-10-10/11 (first-conversation.md sections 5.7 and 6); DEC 4a3b078a-852c-4a4e-87b5-816e17dd0a44.

## Related Artifacts

### Decisions

- DEC 4a3b078a-852c-4a4e-87b5-816e17dd0a44: Derived State and Query Layer: Full Rebuild on Startup, SQLite with sqlite-vec

### Goals

- GOL a72b270f-2caf-4654-bf72-9dfa750f26ea: Support Multi-Hop and Similarity Queries of the Graph
