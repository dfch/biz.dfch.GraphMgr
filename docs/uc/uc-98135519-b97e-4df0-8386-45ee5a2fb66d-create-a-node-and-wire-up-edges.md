---
classification: null
created: '2026-10-13T04:28:22.762+02:00'
id: 98135519-b97e-4df0-8386-45ee5a2fb66d
status: draft
type: uc
updated: '2026-10-13T04:28:22.762+02:00'
version: 1.0.0
---

# Create a Node and Wire Up Edges

## Characteristic Information

### Goal in Context

As a user modeling a graph, I want to create a node of a given type and connect it to existing nodes with typed, directed edges, so that the relationships are recorded, committed, and immediately queryable.

### Scope

GraphMgr, covering the API write path (ABI, REST, or MCP), schema validation, the sharded file storage layout, and the git commit step.

### Level

User Goal

### Preconditions

- A JSON Schema for the requested node type exists under the schemas directory.
- The user can reach the system through one of the three access surfaces.

### Success End Condition

- The node file exists in its sharded node folder and is committed to git.
- Each requested edge file exists in its sharded links path and is committed to git.
- The node and all edges are queryable through the index.

### Failed End Condition

- A schema-validation failure or a dangling edge endpoint produces a structured rejection with no partial write to disk and no git commit.

### Primary Actor

User, through the ABI, REST API, or MCP server.

### Secondary Actors

- git (receives one commit per accepted write)
- Derived query index (absorbs the new entities)

### Trigger

The user issues a create-node call followed by create-edge calls.

### Frequency

The dominant burst workflow: create node, wire up edges, fix mistakes.

### Priority

Top

## Main Success Scenario

1. The user requests node creation with a type and attribute values.
2. GraphMgr validates the attributes against the node type's JSON Schema, allocates a UUID, and writes the node's index file into the sharded node folder.
3. GraphMgr creates exactly one git commit for the write.
4. The user requests edges from the new node to existing nodes.
5. For each edge, GraphMgr validates that both endpoints resolve to existing nodes, validates the edge attributes, allocates a UUID, and writes the edge file into its shard.
6. GraphMgr creates exactly one git commit per edge write.
7. The change-detection pipeline upserts the new entities into the relational index and computes their embeddings.
8. The user queries the node and its edges and receives the committed state.

## Extensions

### Extension 5a. A proposed edge references a node that does not exist

1. GraphMgr rejects the edge with a structured, non-raising error naming the unresolved endpoint.
2. No edge file is written and no git commit is created.
3. Return to step 4 for a corrected edge proposal.

### Extension 8a. The user fixes a mistaken edge

1. The user deletes the edge through the API; GraphMgr removes the edge file and commits.
2. The user creates the replacement edge; a type change is always delete plus create, never an in-place type mutation.
3. Return to step 8.

## Open Issues

- The serialization format of the entity files (YAML/JSON/TOML) is not yet decided.
- The node index-file naming convention (for example `index.<ext>` versus a `node.*` glob) is not yet decided.

## Related Information

### Notes

- REQ 3d4b0381-c825-474b-acaf-3ada75a0ee56: Edge as First-Class Stored Entity
- REQ c7f59a15-31f8-4f10-adc4-becd73deb675: Referential Integrity at Edge Write Time
- REQ 427c23c8-e353-43f2-bb06-b361de9e1e1d: Immutability of Edge Type
- REQ cbad3818-4c88-47ae-ba5b-2f5ae9125e86: Git-Trackable Text Storage for All Entities
- REQ d851888c-2f99-4f84-b0b9-f9de01b55c9b: One Git Commit per API Write
- DEC a6235507-22fa-4aed-a45e-d9707c0f13bd: Storage Layout: One File per Entity, 256 Id-Prefix Shards

### Assumptions

- Each write goes through the single shared core library; no transport-specific logic participates.
- Concurrency on the same entity type is serialized by the in-process write lock.
