---
classification: null
created: '2026-10-13T04:12:13.696+02:00'
id: 6d237d5a-0bab-4bce-b331-51d4a2c008c2
status: draft
type: req
updated: '2026-10-13T04:12:13.696+02:00'
version: 1.0.0
---

# In-Process Write Lock at Type Level

WHILE two or more API writes target the same entity type concurrently, GraphMgr shall serialize them with an in-process write lock at type-level granularity.

## Description

Type-level locking is the starting point: coarser than per-id, simpler and safer. The one-file-per-entity storage design supports finer per-id locking later if ever needed, without changing the storage layout.

## Characteristics

1. Performance Efficiency
2. Reliability

## Level

MUST

## Priority

20

## Tags

- concurrency
- locking

## Source

First design discussion 2026-10-10/11 (first-conversation.md sections 3.4 and 6); DEC 4a3b078a-852c-4a4e-87b5-816e17dd0a44.

## Related Artifacts

### Decisions

- DEC 4a3b078a-852c-4a4e-87b5-816e17dd0a44: Derived State and Query Layer: Full Rebuild on Startup, SQLite with sqlite-vec
