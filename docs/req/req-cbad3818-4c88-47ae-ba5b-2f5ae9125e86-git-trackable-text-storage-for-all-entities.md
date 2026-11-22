---
classification: null
created: '2026-10-13T04:10:03.344+02:00'
id: cbad3818-4c88-47ae-ba5b-2f5ae9125e86
status: draft
type: req
updated: '2026-10-13T04:10:03.344+02:00'
version: 1.0.0
---

# Git-Trackable Text Storage for All Entities

GraphMgr shall store every node and edge entity as a plain-text, git-trackable file; GraphMgr shall not require a binary database for persistence.

## Description

Precise, reviewable change tracking is the core driver of the project: every node and edge must be diffable, blamable, and revertable with standard git tooling, which line-based diffs on binary database formats cannot provide.

## Characteristics

1. Functional Suitability
2. Maintainability

## Level

MUST

## Priority

10

## Tags

- storage
- git

## Source

First design discussion 2026-10-10/11 (first-conversation.md sections 1 and 6); DEC 2c4a32c5-c9a7-4ad7-8740-53288b3ff15b.

## Related Artifacts

### Decisions

- DEC 2c4a32c5-c9a7-4ad7-8740-53288b3ff15b: Overall Approach: Bespoke System Following the specmgr Pattern

### Goals

- GOL 1585066f-9fc7-44bb-a3be-3c5e6cf91cde: Keep Graph Data Precisely Trackable in Git
