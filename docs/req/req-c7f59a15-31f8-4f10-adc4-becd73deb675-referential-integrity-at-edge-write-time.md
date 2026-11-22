---
classification: null
created: '2026-10-13T04:07:10.359+02:00'
id: c7f59a15-31f8-4f10-adc4-becd73deb675
status: draft
type: req
updated: '2026-10-13T04:07:10.359+02:00'
version: 1.0.0
---

# Referential Integrity at Edge Write Time

IF the src or dst of a proposed edge does not resolve to an existing node, THEN GraphMgr shall reject the edge creation.

## Description

Dedicated edge entities (rather than inline reference tags) let the API enforce referential integrity at write time: an edge whose endpoint is missing is never written to disk. This is a deliberate departure from the soft integrity of inline-tag designs, where dangling references are only flagged on read.

## Characteristics

1. Functional Suitability
2. Reliability

## Level

MUST

## Priority

10

## Tags

- data-model
- integrity
- edge

## Source

First design discussion 2026-10-10/11 (first-conversation.md section 3.1); DEC 2c4a32c5-c9a7-4ad7-8740-53288b3ff15b.

## Related Artifacts

### Decisions

- DEC 2c4a32c5-c9a7-4ad7-8740-53288b3ff15b: Overall Approach: Bespoke System Following the specmgr Pattern

### Goals

- GOL 86b457d9-58b0-4c9c-8c53-cf7849a72e75: Provide a Generic, Extensible Typed Node/Edge Graph Model

### Risks

- RSK 6342b261-1cd0-4cdb-8ceb-70c284b8e2dd: Graph Corruption from Out-of-Band Manual Edits
