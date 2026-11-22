---
classification: null
created: '2026-10-13T03:49:36.390+02:00'
id: 1585066f-9fc7-44bb-a3be-3c5e6cf91cde
status: draft
type: gol
updated: '2026-10-13T03:49:36.390+02:00'
version: 1.0.0
---

# Keep Graph Data Precisely Trackable in Git

GraphMgr keeps every node and edge as a git-trackable text file so that all changes to the graph carry precise history, semantic diff, and blame, reviewable with standard git tooling.

## Description

Binary databases are excluded because line-based git diffs on binary blobs are meaningless. The git working tree of text files is the single source of truth; any query or similarity index is a derived, rebuildable cache. The API commits on every write it makes, and direct manual edits plus independent git operations outside the API are also fully supported.

## Priority

10

## Tags

- storage
- git
- audit

## Source

First user message of session ses_ed8c0da78ffeLB78W5hsXJNOor (2026-10-10); git write-path decision recorded in first-conversation.md sections 3.5 and 6.

## More Information

Raises PRB e1dd6083-cbd6-42da-a285-bf9e80c21d5a: Generic Typed Node/Link Graph with Git-Trackable Text Storage.
