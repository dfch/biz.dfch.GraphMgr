---
classification: null
created: '2026-10-13T04:12:25.143+02:00'
id: 0acf8b18-5a02-4a84-a210-c5f634f91fc7
status: draft
type: req
updated: '2026-10-13T04:12:25.143+02:00'
version: 1.0.0
---

# Detection of Out-of-Band Changes Including Uncommitted Edits

WHEN a tracked file is created, modified, or deleted outside the API, including while the change is not staged or committed, GraphMgr shall detect that change and update the derived indexes accordingly.

## Description

Detection must work for the full set of scenarios: uncommitted edit, local commit, pull or merge from a remote, and history rewrite. The user explicitly rejected any mechanism that only sees staged or committed changes; git state must play no role in deciding whether the query index needs updating.

## Characteristics

1. Functional Suitability
2. Reliability

## Level

MUST

## Priority

10

## Tags

- change-detection

## Source

First design discussion 2026-10-10/11 (first-conversation.md section 4); DEC 3cccd4cb-f488-4a6f-8005-f38d2b5e92ef.

## Related Artifacts

### Decisions

- DEC 3cccd4cb-f488-4a6f-8005-f38d2b5e92ef: Change Detection: Git-Independent Filesystem Manifest with Hybrid Watch and Poll

### Goals

- GOL 0503a9d2-0a80-4369-a295-ed978e3e4bef: Detect and Absorb Changes Made Outside the API
