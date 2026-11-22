---
classification: null
created: '2026-10-13T04:06:39.293+02:00'
id: 7eeb13d8-c1bd-45b8-8b24-bca94c2998f9
status: draft
type: req
updated: '2026-10-13T04:06:39.293+02:00'
version: 1.0.0
---

# Node Entity with Core and Type-Specific Attributes

GraphMgr shall store each node as a typed entity carrying the core attributes id (UUID), type, schema_version, created, and updated, plus type-specific attributes validated against the JSON Schema for that node type.

## Description

The core attribute set is fixed across all node types; the type-specific attributes are defined per stereotype and validated generically, so one node record corresponds to what a classic database would keep in a single table row.

## Characteristics

1. Functional Suitability
2. Maintainability

## Level

MUST

## Priority

10

## Tags

- data-model
- node

## Source

First design discussion 2026-10-10/11 (first-conversation.md section 3.1); DEC 2c4a32c5-c9a7-4ad7-8740-53288b3ff15b.

## Related Artifacts

### Decisions

- DEC 2c4a32c5-c9a7-4ad7-8740-53288b3ff15b: Overall Approach: Bespoke System Following the specmgr Pattern

### Goals

- GOL 86b457d9-58b0-4c9c-8c53-cf7849a72e75: Provide a Generic, Extensible Typed Node/Edge Graph Model
