---
classification: null
created: '2026-10-13T03:55:05.899+02:00'
id: 4a3b078a-852c-4a4e-87b5-816e17dd0a44
status: accepted
type: dec
updated: '2026-10-13T04:00:53.021+02:00'
version: 1.0.0
---

# Derived State and Query Layer: Full Rebuild on Startup, SQLite with sqlite-vec

## Context and Problem Statement

The text files are the only source of truth, but multi-hop traversal and similarity search need a query engine. The design therefore maintains derived state: the change-detection manifest, a relational query index, and vector embeddings. Three questions had to be settled: which SQL engine, where the embeddings live, and whether any derived state persists across process restarts. Embedding generation is CPU-bound and non-trivial per item, so "persist nothing, rebuild everything" was in tension with cold-start time at scale.

## Decision Drivers

- A strong query language for multi-hop traversal (recursive CTEs) without standing up a server.
- One engine, one lifecycle: keep the relational index and the vector index together rather than operating a separate vector-DB service (Qdrant/Weaviate/Milvus), consistent with the all-embedded design.
- Self-hosted similarity: local sentence embeddings (fastembed / bge-small class, precedented by specmgr's own find_related / find_similar_text tools), no external API calls.
- A single simple consistency invariant for all derived state.
- Concurrency control and audit: in-process locking around API writes, one git commit per API write, one shared core library so REST and MCP never duplicate business logic.

## Considered Options

SQLite versus DuckDB for the relational index; the sqlite-vec extension (embeddings inside the same SQLite database) versus a dedicated embedded ANN library (faiss/hnswlib) versus an external vector-DB service; persisting the manifest/index/embeddings across restarts versus rebuilding everything from scratch on every start; and type-level versus per-id write-lock granularity as the starting point.

## Decision Outcome

SQLite is the query engine: tables nodes(id, type, created, updated, attrs JSON) and links(id, type, src, dst, directed, created, updated, attrs JSON), with WITH RECURSIVE for multi-hop traversal; harder graph algorithms (weighted shortest path, centrality) can later layer networkx over a materialized subgraph pulled via SQL. Embeddings of each entity's name + description are computed by a small local model and stored via the sqlite-vec extension in the same database; a dedicated ANN library is the fallback if sqlite-vec proves insufficient. The embedding sync piggybacks on the same change-detection pipeline: when an index file's content changes, its embedding is recomputed and upserted alongside the relational row. No derived state persists across restarts: the manifest, the SQLite index, and the embeddings are all rebuilt from scratch on every process start, accepting a slower cold start as the corpus grows. Concurrency uses an in-process write lock at type-level granularity as the starting point (one-file-per-edge supports finer per-id locking later). All behavior lives in one shared core library (the ABI); REST and MCP are thin, logic-free transport wrappers; every API-driven write produces exactly one git commit.

### Consequences

- Cold start grows with the corpus; embedding recomputation is the first genuinely slow part at scale (stat/hash checks stay sub-millisecond). The "rebuild everything, persist nothing" rule keeps one consistency invariant and makes any suspected index simply nukable.
- One engine, one lifecycle for relational and vector queries; similarity results can be joined with normal SQL/graph filters (e.g. similar nodes, but only of type X).
- Type-level locking is coarser than necessary (one-file-per-edge could lock per id); this is an accepted starting point, not a final constraint.
- The serialization format of the entity files, the programming language, and the watcher library remain open choices that this decision does not constrain.

### Confirmation

The user chose SQLite over DuckDB, confirmed the rebuild-from-scratch policy for the manifest, and extended it to the SQLite index and the vector-embedding index (rejecting a persistent embedding cache) during the 2026-10-10/11 design discussion (first-conversation.md sections 3.3, 3.4, 5.7, and 6).

## Roles and Responsibilities

### Accountable

The project owner (user of the 2026-10-10/11 design discussion).

### Responsible

- The GraphMgr development team (to be assigned at implementation start).

### Consulted

- The 2026-10-10/11 design discussion, including the sqlite-vec versus external vector-DB trade-off and the embedding-cache policy question.

### Informed

- All future maintainers of the GraphMgr specification documents.

## Tags

- query
- sqlite
- embeddings
- concurrency

## Source

Session ses_ed8c0da78ffeLB78W5hsXJNOor, recorded in first-conversation.md sections 3.3, 3.4, 5.7, and 6 and in session-first-conversation-ses_ed8c.md (both at the repo root).

## Related Artifacts

### Goals

- GOL a72b270f-2caf-4654-bf72-9dfa750f26ea: Support Multi-Hop and Similarity Queries of the Graph
- GOL 0503a9d2-0a80-4369-a295-ed978e3e4bef: Detect and Absorb Changes Made Outside the API

## Pros and Cons

### Option 1: SQLite + sqlite-vec, no persisted derived state, rebuild everything on start (chosen)

Pros: stdlib/ubiquitous engine with solid WITH RECURSIVE support; one engine and one lifecycle for relational and vector queries; self-hosted embeddings with no external API calls; a single simple consistency invariant (process start = guaranteed-correct full resync); any index can be nuked and rebuilt from the text files. Cons: cold start grows with corpus size, and re-embedding everything on every restart is the first genuinely slow part at scale.

### Option 2: DuckDB + external vector-DB service + persisted derived caches

Pros: richer native JSON/analytical performance; a production-grade vector service at very large scale; fast cold starts if caches are persisted. Cons: two (or more) engines and lifecycles to operate, inconsistent with the all-embedded design; persisted caches reintroduce exactly the staleness problem the rebuild rule removes; heavier deployment footprint than the project needs today.

## Updates

<!-- Newest entry first -- prepend new entries directly below this comment. -->

### 2026-10-13 03:52:00.000+02:00 - Created

Decision recorded from the 2026-10-10/11 design discussion; SQLite, sqlite-vec, and the full-rebuild-on-startup policy were confirmed by the user.
