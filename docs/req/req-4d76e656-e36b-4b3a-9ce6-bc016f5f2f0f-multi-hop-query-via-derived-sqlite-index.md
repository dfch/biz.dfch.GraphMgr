---
classification: null
created: '2026-10-13T04:14:34.207+02:00'
id: 4d76e656-e36b-4b3a-9ce6-bc016f5f2f0f
status: draft
type: req
updated: '2026-10-13T04:14:34.207+02:00'
version: 1.0.0
---

# Multi-Hop Query via Derived SQLite Index

GraphMgr shall support multi-hop graph traversal queries against a SQLite relational index with nodes and links tables (core columns plus a JSON column for type-specific attributes); GraphMgr shall treat that index as a derived, rebuildable cache, and the git working tree of text files shall be the sole source of truth.

## Description

Text files are weak at multi-hop queries, so the index loads them into SQLite, which has solid WITH RECURSIVE support for traversal. Harder graph algorithms (weighted shortest path, centrality) can later layer networkx over a materialized subgraph pulled via SQL. If the index is ever suspect, it can be deleted and rebuilt from disk; it is never authoritative.

## Characteristics

1. Functional Suitability
2. Performance Efficiency

## Level

MUST

## Priority

10

## Tags

- query
- sqlite
- multi-hop

## Source

First design discussion 2026-10-10/11 (first-conversation.md sections 2, 3.3, and 6); DEC 4a3b078a-852c-4a4e-87b5-816e17dd0a44.

## Related Artifacts

### Decisions

- DEC 4a3b078a-852c-4a4e-87b5-816e17dd0a44: Derived State and Query Layer: Full Rebuild on Startup, SQLite with sqlite-vec

### Goals

- GOL a72b270f-2caf-4654-bf72-9dfa750f26ea: Support Multi-Hop and Similarity Queries of the Graph
