---
classification: null
created: '2026-10-13T04:10:16.816+02:00'
id: 41498f3b-3061-43d1-b910-44cc5d859409
status: draft
type: req
updated: '2026-10-13T04:10:16.816+02:00'
version: 1.0.0
---

# Node Folder Layout with Attachments

GraphMgr shall store each node in a folder `nodes/<type>/<id[0:2]>/<id>/` containing a convention-named index file with the node's attributes and optional sibling attachment files; GraphMgr shall treat attachment files as opaque payload, served on request, not parsed into the query index.

## Description

Nodes must be able to hold additional data files. Folder-per-node keeps the node/edge relationship unchanged while allowing arbitrary sibling attachments. Only the convention-named index file feeds the query index. This layout is precedented by the folder-per-entity design used for the one specmgr domain that needs multiple files.

## Characteristics

1. Functional Suitability
2. Maintainability

## Level

MUST

## Priority

10

## Tags

- storage
- node
- attachments

## Source

First design discussion 2026-10-10/11 (first-conversation.md section 5.5); DEC a6235507-22fa-4aed-a45e-d9707c0f13bd.

## Related Artifacts

### Decisions

- DEC a6235507-22fa-4aed-a45e-d9707c0f13bd: Storage Layout: One File per Entity, 256 Id-Prefix Shards

### Goals

- GOL 1585066f-9fc7-44bb-a3be-3c5e6cf91cde: Keep Graph Data Precisely Trackable in Git

### Risks

- RSK 380eb6dc-eb38-4046-8884-14b4058d6410: Inode Exhaustion on Fixed-Inode-Count Filesystems
