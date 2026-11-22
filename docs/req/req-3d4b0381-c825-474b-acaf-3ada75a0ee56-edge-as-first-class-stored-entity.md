---
classification: null
created: '2026-10-13T04:06:57.981+02:00'
id: 3d4b0381-c825-474b-acaf-3ada75a0ee56
status: draft
type: req
updated: '2026-10-13T04:06:57.981+02:00'
version: 1.0.0
---

# Edge as First-Class Stored Entity

GraphMgr shall store each edge as a first-class typed entity separate from any node file, carrying the core attributes id, type, src, dst, directed, schema_version, created, and updated, plus type-specific attributes validated against the JSON Schema for that edge type.

## Description

Edges are not inline references inside node files. They are their own records with UML-connector semantics: typed, directed (src, dst), and able to carry their own attributes such as label, multiplicity, navigability, or weight.

## Characteristics

1. Functional Suitability

## Level

MUST

## Priority

10

## Tags

- data-model
- edge

## Source

First design discussion 2026-10-10/11 (first-conversation.md sections 2 and 3.1); DEC 2c4a32c5-c9a7-4ad7-8740-53288b3ff15b.

## Related Artifacts

### Decisions

- DEC 2c4a32c5-c9a7-4ad7-8740-53288b3ff15b: Overall Approach: Bespoke System Following the specmgr Pattern

### Goals

- GOL 86b457d9-58b0-4c9c-8c53-cf7849a72e75: Provide a Generic, Extensible Typed Node/Edge Graph Model
