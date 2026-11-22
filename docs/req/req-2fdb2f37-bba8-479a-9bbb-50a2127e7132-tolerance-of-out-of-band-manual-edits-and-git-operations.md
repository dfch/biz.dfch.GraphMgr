---
classification: null
created: '2026-10-13T04:12:04.944+02:00'
id: 2fdb2f37-bba8-479a-9bbb-50a2127e7132
status: draft
type: req
updated: '2026-10-13T04:12:04.944+02:00'
version: 1.0.0
---

# Tolerance of Out-of-Band Manual Edits and Git Operations

GraphMgr shall tolerate direct manual edits and independent git operations (commit, pull, merge, rebase) made outside the API.

## Description

The git write path is dual-mode: the API commits on every write it makes, and the repository must also tolerate hand-edited files and independent git pushes, pulls, and merges. This is a confirmed user requirement and the reason change detection had to be filesystem-level rather than git-state-based.

## Characteristics

1. Functional Suitability
2. Flexibility

## Level

MUST

## Priority

10

## Tags

- git
- change-detection

## Source

First design discussion 2026-10-10/11 (first-conversation.md sections 3.5 and 6); DEC 3cccd4cb-f488-4a6f-8005-f38d2b5e92ef.

## Related Artifacts

### Decisions

- DEC 3cccd4cb-f488-4a6f-8005-f38d2b5e92ef: Change Detection: Git-Independent Filesystem Manifest with Hybrid Watch and Poll

### Goals

- GOL 0503a9d2-0a80-4369-a295-ed978e3e4bef: Detect and Absorb Changes Made Outside the API
