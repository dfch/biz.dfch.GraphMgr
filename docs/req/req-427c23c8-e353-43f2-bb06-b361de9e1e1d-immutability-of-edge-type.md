---
classification: null
created: '2026-10-13T04:07:19.192+02:00'
id: 427c23c8-e353-43f2-bb06-b361de9e1e1d
status: draft
type: req
updated: '2026-10-13T04:07:19.192+02:00'
version: 1.0.0
---

# Immutability of Edge Type

GraphMgr shall not change the type of an existing edge in place; a type change shall be performed as deletion of the old edge and creation of a new edge.

## Description

Edge type is immutable once created. This simplifies schema handling: an existing edge record never needs re-validation against a different type's schema.

## Characteristics

1. Functional Suitability

## Level

MUST

## Priority

20

## Tags

- data-model
- edge

## Source

First design discussion 2026-10-10/11 (first-conversation.md section 5.2); DEC a6235507-22fa-4aed-a45e-d9707c0f13bd.

## Related Artifacts

### Decisions

- DEC a6235507-22fa-4aed-a45e-d9707c0f13bd: Storage Layout: One File per Entity, 256 Id-Prefix Shards

### Goals

- GOL 86b457d9-58b0-4c9c-8c53-cf7849a72e75: Provide a Generic, Extensible Typed Node/Edge Graph Model
