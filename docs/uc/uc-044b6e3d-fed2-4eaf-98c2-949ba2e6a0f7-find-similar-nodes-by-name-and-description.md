---
classification: null
created: '2026-10-13T04:24:19.548+02:00'
id: 044b6e3d-fed2-4eaf-98c2-949ba2e6a0f7
status: draft
type: uc
updated: '2026-10-13T04:24:19.548+02:00'
version: 1.0.0
---

# Find Similar Nodes by Name and Description

## Characteristic Information

### Goal in Context

As a user, I want to find nodes or edges that are semantically similar to a query (or to another entity), based on their name and description, so that I can discover related entities without knowing their exact identifiers.

### Scope

GraphMgr's vector-similarity path: local embedding computation, the sqlite-vec store inside the shared SQLite database, and the query surface (ABI, REST, or MCP).

### Level

User Goal

### Preconditions

- The process has computed embeddings during its startup rebuild (or after the relevant entities were synced).

### Success End Condition

- The query returns the top-K most similar entities with similarity scores, optionally filtered by entity type.

### Failed End Condition

- If embeddings are not yet available (startup rebuild in progress), the query surface reports the system as not ready rather than returning an empty or partial result silently.

### Primary Actor

User, through the ABI, REST API, or MCP server.

### Secondary Actors

- Local embedding model (no external API calls)
- sqlite-vec extension inside the shared SQLite database

### Trigger

The user submits a similarity query (free text, or an existing entity to compare against), a K value, and an optional type filter.

### Frequency

Occasional; supporting discovery rather than the primary read workload.

## Main Success Scenario

1. The user submits a similarity query with free text, K, and an optional type filter.
2. GraphMgr embeds the query text with the local model.
3. GraphMgr runs a nearest-neighbor search over the stored embeddings via sqlite-vec.
4. GraphMgr applies the optional type filter and returns the top-K entities with their similarity scores.

## Extensions

### Extension 2a. The query references an existing entity instead of free text

1. GraphMgr reuses the entity's stored embedding instead of recomputing one.
2. Return to step 3.

## Open Issues

- Cold-start latency for embedding a large corpus is accepted but not yet benchmarked (tracked as a risk).
- The embedding model and vector dimensions are implementation choices, not yet fixed.

## Related Information

### Notes

- REQ 64d52fab-3260-4d96-aefe-a2e2aa4ad11f: Similarity Search over Name and Description
- REQ 06f32855-c050-4fcd-8538-a3550ccff070: Full Rebuild of Derived State on Process Start
- RSK 8401d706-a3dd-4205-9675-f73daec6d144: Cold-Start Degradation from Full Rebuild and Full Re-Embedding
- DEC 4a3b078a-852c-4a4e-87b5-816e17dd0a44: Derived State and Query Layer: Full Rebuild on Startup, SQLite with sqlite-vec

### Assumptions

- The entity's schema includes name and description fields that feed the embedding.
- Embeddings are a derived, rebuildable index: they are recomputed on every startup and on detected content changes, and never persisted across restarts.
