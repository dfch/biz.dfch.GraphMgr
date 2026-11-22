---
classification: null
created: '2026-10-13T04:10:30.730+02:00'
id: e40ca5ca-23be-4d13-b0dc-2012548312b5
status: draft
type: req
updated: '2026-10-13T04:10:30.730+02:00'
version: 1.0.0
---

# Edge File Layout Sharded by Own Id

GraphMgr shall store each edge as a single file `links/<type>/<id[0:2]>/<id>.<ext>`; GraphMgr shall derive the shard directory of an edge from the first two characters of the edge's own id, not from src or dst.

## Description

Both nodes and edges are directory-sharded by 256 shard directories (id prefix, two hex characters). Sharding edges by their own random id, never by endpoint, spreads a single node's burst of edge creation evenly across shards instead of clustering it into one shard; sharding by endpoint would be the worst case for contention exactly during the confirmed create-node-then-wire-up-edges workflow.

## Characteristics

1. Functional Suitability
2. Performance Efficiency

## Level

MUST

## Priority

10

## Tags

- storage
- edge
- sharding

## Source

First design discussion 2026-10-10/11 (first-conversation.md sections 5.2 and 5.5); DEC a6235507-22fa-4aed-a45e-d9707c0f13bd.

## Related Artifacts

### Decisions

- DEC a6235507-22fa-4aed-a45e-d9707c0f13bd: Storage Layout: One File per Entity, 256 Id-Prefix Shards

### Goals

- GOL 1585066f-9fc7-44bb-a3be-3c5e6cf91cde: Keep Graph Data Precisely Trackable in Git
