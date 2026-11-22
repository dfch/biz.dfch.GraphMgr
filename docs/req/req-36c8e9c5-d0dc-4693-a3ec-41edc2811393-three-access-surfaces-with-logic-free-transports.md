---
classification: null
created: '2026-10-13T04:11:43.043+02:00'
id: 36c8e9c5-d0dc-4693-a3ec-41edc2811393
status: draft
type: req
updated: '2026-10-13T04:11:43.043+02:00'
version: 1.0.0
---

# Three Access Surfaces with Logic-Free Transports

GraphMgr shall expose all functionality through a shared core library (ABI), a REST API, and an MCP server; the REST and MCP layers shall be thin transport wrappers that contain no duplicated business logic.

## Description

One shared core library implements validation, locking, file I/O, git commit, index sync, and queries. All three surfaces (library, REST, MCP) wrap that single core, following the same discipline as the specmgr system. No transport may implement business logic of its own.

## Characteristics

1. Functional Suitability
2. Maintainability

## Level

MUST

## Priority

10

## Tags

- api
- abi
- architecture

## Source

First design discussion 2026-10-10/11 (first-conversation.md sections 1, 3.4, and 6); DEC 4a3b078a-852c-4a4e-87b5-816e17dd0a44.

## Related Artifacts

### Decisions

- DEC 4a3b078a-852c-4a4e-87b5-816e17dd0a44: Derived State and Query Layer: Full Rebuild on Startup, SQLite with sqlite-vec

### Goals

- GOL 86b457d9-58b0-4c9c-8c53-cf7849a72e75: Provide a Generic, Extensible Typed Node/Edge Graph Model
