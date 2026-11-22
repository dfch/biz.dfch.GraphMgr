---
classification: null
created: '2026-10-13T04:18:28.437+02:00'
id: 7f0c025c-4469-4f99-a310-861fd3239789
status: draft
type: vcr
updated: '2026-10-13T04:18:28.437+02:00'
version: 1.0.0
---

# Verification of Out-of-Band Change Detection Including Uncommitted Edits

## Verifies

REQ 0acf8b18-5a02-4a84-a210-c5f634f91fc7: Detection of Out-of-Band Changes Including Uncommitted Edits

This record defines how the core change-detection guarantee is verified: files created, modified, or deleted outside the API, including while not staged or committed, must be detected and reflected in the derived indexes without a process restart. Detection must be filesystem-based (git-independent), per the accepted change-detection decision.

## Coverage

partial

## Acceptance Criteria

### AC-001 (Test): Uncommitted in-place edit is reflected within one poll interval plus debounce

An edit made directly to an existing node's index file without any git staging or commit appears in API query results within one poll interval plus the debounce window.

#### Test Steps

1. Start the API process against a populated tree and wait for initial sync.
2. Modify a type-specific attribute of an existing node directly on disk; run no git commands.
3. Wait one configured poll interval plus the debounce window.
4. Query the node through the API and assert that the new attribute value is returned.

### AC-002 (Test): Out-of-band file creation and deletion are reflected

A schema-valid node file created directly on disk (uncommitted) appears in queries; deleting the file removes the node from queries, both without a restart.

#### Test Steps

1. Write a schema-valid node index file directly into the correct sharded node folder; run no git commands.
2. Wait one poll interval plus the debounce window and assert that the node is queryable.
3. Delete the node folder directly.
4. Wait one poll interval plus the debounce window and assert that the node no longer appears in queries.

### AC-003 (Demonstration): git pull of new entities is absorbed while running

Running git pull of a branch that adds entities while the API process is running causes the new entities to become queryable without a restart.

#### Test Steps

1. Start the API process against a populated tree.
2. In a separate checkout, create a branch with new node and edge files and merge it into the working tree with git pull or git merge.
3. Wait for the next detection cycle.
4. Assert that the new entities are queryable through the API and that their commit is visible in git log.

### AC-004 (Inspection): Detection is filesystem-based, not git-state-based

Source inspection confirms that the sync decision derives from the in-memory manifest (mtime/size prefilter, content hash) and that no git command output (status, diff, HEAD SHA) feeds the change-detection path.

## More Information

Drafted 2026-10-13 as part of the initial specification set; the four-scenario model (uncommitted edit, local commit, pull/merge, history rewrite) from the design discussion is covered across AC-001 to AC-003.

## Updates

<!-- Newest entry first -- prepend new entries directly below this comment. -->

### 2026-10-13 04:15:00.000+02:00 - Created

Verification case drafted from the accepted change-detection decision; coverage is none pending implementation.
