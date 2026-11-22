---
classification: null
created: '2026-10-13T04:14:06.574+02:00'
id: d20e1a61-7dad-4978-812b-53f35287acf4
status: draft
type: req
updated: '2026-10-13T04:14:06.574+02:00'
version: 1.0.0
---

# Per-File Parse and Validation Failure Isolation

IF a file fails to parse or fails schema validation, THEN GraphMgr shall mark that single entity as invalid without blocking synchronization of any other entity, and shall retry that file on its next detected change.

## Description

A watch event can fire while a file is only half-written, and hand edits can be malformed. Parse or validation failure is a non-raising, per-file condition: the affected entity is marked invalid or stale in the index, everything else syncs normally, and no process crash results. This is the same discipline the specmgr system applies to its structured error results.

## Characteristics

1. Reliability

## Level

MUST

## Priority

10

## Tags

- change-detection
- reliability

## Source

First design discussion 2026-10-10/11 (first-conversation.md section 4.3); DEC 3cccd4cb-f488-4a6f-8005-f38d2b5e92ef.

## Related Artifacts

### Decisions

- DEC 3cccd4cb-f488-4a6f-8005-f38d2b5e92ef: Change Detection: Git-Independent Filesystem Manifest with Hybrid Watch and Poll

### Risks

- RSK 6342b261-1cd0-4cdb-8ceb-70c284b8e2dd: Graph Corruption from Out-of-Band Manual Edits
