---
classification: null
created: '2026-10-13T04:11:53.806+02:00'
id: d851888c-2f99-4f84-b0b9-f9de01b55c9b
status: draft
type: req
updated: '2026-10-13T04:11:53.806+02:00'
version: 1.0.0
---

# One Git Commit per API Write

WHEN GraphMgr performs a write through the API, GraphMgr shall create exactly one git commit for that write.

## Description

Git commits form the audit trail of API-driven changes: one commit per logical write gives maximal history granularity for git log and git blame. Direct manual edits and independent git operations outside the API are a separate, supported path (see the tolerance requirement).

## Characteristics

1. Functional Suitability
2. Reliability

## Level

MUST

## Priority

20

## Tags

- api
- git
- audit

## Source

First design discussion 2026-10-10/11 (first-conversation.md sections 3.4 and 6); DEC 4a3b078a-852c-4a4e-87b5-816e17dd0a44.

## Related Artifacts

### Decisions

- DEC 4a3b078a-852c-4a4e-87b5-816e17dd0a44: Derived State and Query Layer: Full Rebuild on Startup, SQLite with sqlite-vec

### Goals

- GOL 1585066f-9fc7-44bb-a3be-3c5e6cf91cde: Keep Graph Data Precisely Trackable in Git
