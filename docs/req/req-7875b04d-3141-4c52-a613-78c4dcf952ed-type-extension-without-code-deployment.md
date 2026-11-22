---
classification: null
created: '2026-10-13T04:09:43.019+02:00'
id: 7875b04d-3141-4c52-a613-78c4dcf952ed
status: draft
type: req
updated: '2026-10-13T04:09:43.019+02:00'
version: 1.0.0
---

# Type Extension without Code Deployment

WHEN a new node or edge type is introduced, GraphMgr shall enable that type by adding a JSON Schema file under the schemas directory, without requiring deployment of new code.

## Description

Type schemas are data-driven JSON Schema files (schemas/node/ and schemas/edge/), validated generically. Adding a stereotype is a configuration change, not a code change, which is what keeps the graph model generic and extensible.

## Characteristics

1. Functional Suitability
2. Maintainability

## Level

MUST

## Priority

10

## Tags

- data-model
- extensibility

## Source

First design discussion 2026-10-10/11 (first-conversation.md section 3.2); DEC 2c4a32c5-c9a7-4ad7-8740-53288b3ff15b.

## Related Artifacts

### Decisions

- DEC 2c4a32c5-c9a7-4ad7-8740-53288b3ff15b: Overall Approach: Bespoke System Following the specmgr Pattern

### Goals

- GOL 86b457d9-58b0-4c9c-8c53-cf7849a72e75: Provide a Generic, Extensible Typed Node/Edge Graph Model
