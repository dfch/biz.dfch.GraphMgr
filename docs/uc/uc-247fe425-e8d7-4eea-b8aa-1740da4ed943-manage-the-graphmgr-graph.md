---
classification: null
created: '2026-10-13T05:01:12.748+02:00'
id: 247fe425-e8d7-4eea-b8aa-1740da4ed943
status: draft
type: uc
updated: '2026-10-13T05:01:12.748+02:00'
version: 1.0.0
---

# Manage the GraphMgr Graph

## Characteristic Information

### Goal in Context

As a user or developer, I want to build, keep in sync, and explore a graph of typed nodes and edges in GraphMgr, so that the graph stays accurate, git-trackable, and queryable through direct API writes, out-of-band edits, multi-hop traversal, and similarity search.

### Scope

GraphMgr as a whole: the API write path, the out-of-band filesystem change-detection path, and the derived SQLite relational index and sqlite-vec similarity layers.

### Level

Summary

### Preconditions

- GraphMgr is running with its API, change-detection watcher, and derived indexes initialized.
- JSON Schemas for the node and edge types in use already exist under the schemas directory.

### Success End Condition

- The graph's nodes and edges exist as git-committed files and are reflected in the derived indexes.
- The graph can be queried by multi-hop traversal and by name/description similarity, and returns current, consistent results.

### Failed End Condition

- A write is rejected (schema-validation failure, dangling edge, or invalid out-of-band file) without corrupting the rest of the graph or leaving a partial commit.
- A query reports the system as not ready, rather than returning stale or partial results silently, while the index is rebuilding.

### Primary Actor

User or developer, through the ABI, REST API, MCP server, a text editor, or git.

### Secondary Actors

- Local embedding model (for similarity queries)
- sqlite-vec and the derived SQLite relational index
- git (receives one commit per accepted write and carries out-of-band changes)
- Filesystem watcher and periodic poll (for out-of-band change detection)

### Trigger

A user or developer wants to add to, change, or ask a question of the graph, through any of the supported access paths.

### Frequency

Continuous; this summary covers the system's dominant read and write workflows.

### Priority

Top

### Related Use Cases

- Subordinate: Create a Node and Wire Up Edges (UC 98135519-b97e-4df0-8386-45ee5a2fb66d)
- Subordinate: Edit Graph Data Outside the API (UC 0b1ae5ce-bc8e-4e11-beb8-3b75455dfec8)
- Subordinate: Query the Graph with a Multi-Hop Traversal (UC 7f0637c2-9989-4d06-b1de-7c8a402aa17b)
- Subordinate: Find Similar Nodes by Name and Description (UC 044b6e3d-fed2-4eaf-98c2-949ba2e6a0f7)

## Main Success Scenario

1. The user or developer creates a node and wires up its edges through the API (UC 98135519-b97e-4df0-8386-45ee5a2fb66d).
2. The developer may instead, or additionally, edit graph data directly on disk or through git, outside the API (UC 0b1ae5ce-bc8e-4e11-beb8-3b75455dfec8).
3. GraphMgr's change-detection pipeline reconciles any out-of-band edits with the derived indexes, so both write paths converge to the same consistent state.
4. The user queries the graph with a multi-hop traversal to answer reachability and dependency-closure questions (UC 7f0637c2-9989-4d06-b1de-7c8a402aa17b).
5. The user finds nodes or edges similar to a query by name and description, when exact identifiers are not known (UC 044b6e3d-fed2-4eaf-98c2-949ba2e6a0f7).
6. The user repeats any of steps 1, 2, 4, or 5 as the graph and its usage evolve.

## Extensions

Each subordinate use case carries its own extensions and failure handling; see the referenced use cases for detail. At the summary level:

### Extension 1a. The write is rejected

1. GraphMgr returns a structured rejection (schema-validation failure, dangling edge endpoint, or per-file validation failure) without a partial write or commit.
2. The user or developer corrects the input and retries the applicable subordinate use case.

### Extension 4a. The index or embeddings are not yet available

1. GraphMgr reports the system as not ready, instead of returning an empty, partial, or stale result.
2. The user retries once the startup rebuild or pending synchronization completes.

## Open Issues

- The dangling-edge policy after an out-of-band node deletion is not yet resolved (see UC 0b1ae5ce-bc8e-4e11-beb8-3b75455dfec8).
- Read-path consistency semantics while an out-of-band change is mid-sync are not yet resolved (see UC 7f0637c2-9989-4d06-b1de-7c8a402aa17b).
- The entity file serialization format and node index-file naming convention are not yet decided (see UC 98135519-b97e-4df0-8386-45ee5a2fb66d).
- The embedding model, vector dimensions, and cold-start re-embedding cost are not yet fixed (see UC 044b6e3d-fed2-4eaf-98c2-949ba2e6a0f7).

## Related Information

### Notes

- This is a summary use case that encompasses the four GraphMgr use cases listed under Related Use Cases.
- GOL 86b457d9-58b0-4c9c-8c53-cf7849a72e75: Provide a Generic, Extensible Typed Node/Edge Graph Model
- GOL 0503a9d2-0a80-4369-a295-ed978e3e4bef: Detect and Absorb Changes Made Outside the API
- GOL a72b270f-2caf-4654-bf72-9dfa750f26ea: Support Multi-Hop and Similarity Queries of the Graph
- GOL 1585066f-9fc7-44bb-a3be-3c5e6cf91cde: Keep Graph Data Precisely Trackable in Git

### Assumptions

- The git working tree of text files remains the sole source of truth; derived indexes (SQLite, sqlite-vec) are rebuildable caches, never authoritative.
- All write paths funnel through the single shared core library, so API writes and out-of-band edits converge to the same validated, indexed state.
