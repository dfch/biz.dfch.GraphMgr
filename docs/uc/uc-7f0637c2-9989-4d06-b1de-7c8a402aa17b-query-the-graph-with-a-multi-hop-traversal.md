---
classification: null
created: '2026-10-13T04:23:48.253+02:00'
id: 7f0637c2-9989-4d06-b1de-7c8a402aa17b
status: draft
type: uc
updated: '2026-10-13T04:23:48.253+02:00'
version: 1.0.0
---

# Query the Graph with a Multi-Hop Traversal

## Characteristic Information

### Goal in Context

As a user or downstream system, I want to traverse the graph beyond direct neighbors (for example, all nodes reachable within N hops from a start node, optionally filtered by type), so that I can answer dependency-closure and reachability questions with a strong query language.

### Scope

GraphMgr's query path: the derived SQLite relational index (nodes and links tables) and the query surface (ABI, REST, or MCP).

### Level

User Goal

### Preconditions

- The process has completed its startup rebuild and the index reflects the current tree (or is within one detection cycle of it).

### Success End Condition

- The query returns the correct reachable set with hop counts and paths, computed from the SQLite index.

### Failed End Condition

- If the index is unavailable (for example, still rebuilding at startup), the query surface reports the system as not ready rather than returning stale or partial results silently.

### Primary Actor

User or downstream system, through the ABI, REST API, or MCP server.

### Secondary Actors

- SQLite relational index (derived, rebuildable cache)

### Trigger

The user submits a traversal query with a start node, a maximum hop count, and optional type filters.

### Frequency

Regular; the primary read workload alongside simple lookups.

### Performance Target

Traversal results return from the in-memory index, not from a re-scan of the text files.

## Main Success Scenario

1. The user submits a multi-hop traversal query (start node, max hops, optional type filters).
2. GraphMgr executes the traversal against the SQLite links table using recursive CTEs.
3. GraphMgr returns the reachable nodes with hop counts and the traversed paths.
4. The user uses the result set, optionally joined with type-specific attribute filters.

## Extensions

### Extension 2a. An out-of-band change is mid-sync when the query arrives

1. GraphMgr serves the query from the last consistent index state and lets the detection pipeline continue in the background.
2. A subsequent query reflects the change once synchronization has caught up.

### Extension 4a. The result set is needed for a harder graph algorithm

1. GraphMgr materializes the relevant subgraph via SQL for an algorithm layer (for example networkx) instead of forcing the algorithm into recursive CTEs.
2. Return the algorithm result to the user.

## Open Issues

- The exact read-path consistency semantics while an out-of-change is mid-sync (serve last consistent state versus wait for sync) is not yet resolved.

## Related Information

### Notes

- REQ 4d76e656-e36b-4b3a-9ce6-bc016f5f2f0f: Multi-Hop Query via Derived SQLite Index
- REQ 0acf8b18-5a02-4a84-a210-c5f634f91fc7: Detection of Out-of-Band Changes Including Uncommitted Edits
- DEC 4a3b078a-852c-4a4e-87b5-816e17dd0a44: Derived State and Query Layer: Full Rebuild on Startup, SQLite with sqlite-vec

### Assumptions

- The git working tree of text files remains the sole source of truth; the index is never authoritative.
- Core columns (id, type, src, dst, timestamps) are proper SQL columns; type-specific attributes live in a JSON column.
