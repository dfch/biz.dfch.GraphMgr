---
classification: null
created: '2026-10-13T03:49:53.960+02:00'
id: 0503a9d2-0a80-4369-a295-ed978e3e4bef
status: draft
type: gol
updated: '2026-10-13T03:49:53.960+02:00'
version: 1.0.0
---

# Detect and Absorb Changes Made Outside the API

GraphMgr detects changes to the data made directly in the working tree, including edits that are not staged or committed, and reflects them in query results without a process restart, without corrupting the rest of the graph.

## Description

Because direct manual editing and independent git operations are a supported write path (see the git write-path decision), change detection must be pure filesystem content tracking, completely decoupled from git. Git serves as the history/audit layer only. A single bad or mid-write file must never block synchronization of everything else.

## Priority

20

## Tags

- change-detection
- reliability

## Source

User correction in session ses_ed8c0da78ffeLB78W5hsXJNOor (detection must work even when changes are not staged or committed); final design recorded in first-conversation.md sections 4 and 6.

## More Information

Raises PRB e1dd6083-cbd6-42da-a285-bf9e80c21d5a: Generic Typed Node/Link Graph with Git-Trackable Text Storage.
