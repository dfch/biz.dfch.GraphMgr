---
classification: null
created: '2026-10-13T04:13:39.357+02:00'
id: 96bbe6fb-e360-4a74-8dc4-2561aa2cd6c5
status: draft
type: req
updated: '2026-10-13T04:13:39.357+02:00'
version: 1.0.0
---

# Git-Independent Change-Detection Basis

GraphMgr shall determine whether a file changed from filesystem content tracking (mtime and size prefilter, then content hash); GraphMgr shall not use git state as the basis for change detection.

## Description

The change-detection manifest maps each tracked path to mtime, size, and content hash. A cheap stat check pre-filters before paying for a full content hash; a fast non-cryptographic hash (xxHash-class) is sufficient because this is change detection, not a security use case. Git is demoted to a history/audit layer only.

## Characteristics

1. Functional Suitability
2. Flexibility

## Level

MUST

## Priority

10

## Tags

- change-detection

## Source

First design discussion 2026-10-10/11 (first-conversation.md sections 4.2 and 4.3); DEC 3cccd4cb-f488-4a6f-8005-f38d2b5e92ef.

## Related Artifacts

### Decisions

- DEC 3cccd4cb-f488-4a6f-8005-f38d2b5e92ef: Change Detection: Git-Independent Filesystem Manifest with Hybrid Watch and Poll

### Goals

- GOL 0503a9d2-0a80-4369-a295-ed978e3e4bef: Detect and Absorb Changes Made Outside the API
