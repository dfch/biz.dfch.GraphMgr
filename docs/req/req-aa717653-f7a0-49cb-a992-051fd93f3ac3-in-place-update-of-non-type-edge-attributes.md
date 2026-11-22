---
classification: null
created: '2026-10-13T04:09:53.462+02:00'
id: aa717653-f7a0-49cb-a992-051fd93f3ac3
status: draft
type: req
updated: '2026-10-13T04:09:53.462+02:00'
version: 1.0.0
---

# In-Place Update of Non-Type Edge Attributes

WHEN an attribute of an edge other than type is changed through the API, GraphMgr shall update that attribute in place.

## Description

Non-type edge attributes (for example label or weight) do get updated in place; this is a real, recurring cost of the confirmed workload, not a rare case. In-place updates must cost O(1), independent of total edge count.

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
