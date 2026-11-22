---
classification: null
created: '2026-10-13T04:25:07.686+02:00'
id: 0b1ae5ce-bc8e-4e11-beb8-3b75455dfec8
status: draft
type: uc
updated: '2026-10-13T04:25:07.686+02:00'
version: 1.0.0
---

# Edit Graph Data Outside the API

## Characteristic Information

### Goal in Context

As a developer, I want to edit entity files directly in my editor or bring changes in through git (pull, merge) without going through the API, so that manual fixes, bulk imports, and review-driven edits work naturally, and the running system picks them up without a restart.

### Scope

GraphMgr's change-detection path: the in-memory filesystem manifest, the shard-level OS watches, the periodic poll, per-file validation, and the index upserts.

### Level

User Goal

### Preconditions

- The API process is running with its watchdog and poll active.
- The edited content targets the convention-named index files (for nodes, the attachments are opaque and not indexed).

### Success End Condition

- The change (create, modify, or delete) is reflected in the derived indexes and in API query results, without a process restart.

### Failed End Condition

- An invalid or mid-write file is marked as an isolated per-file condition: it does not block synchronization of any other entity, and the system stays up.

### Primary Actor

Developer, with a text editor and git, outside the API.

### Secondary Actors

- Filesystem (watch events and poll observations)
- git (history/audit only; plays no role in change detection)

### Trigger

A tracked file is created, modified, or deleted on disk, staged or not.

### Frequency

Supported path, used for manual fixes and bulk operations; most day-to-day writes go through the API.

## Main Success Scenario

1. The developer modifies, creates, or deletes entity files directly (no git add, no git commit).
2. The shard-level watchdog fires (debounced) and/or the periodic poll sweeps the tree.
3. GraphMgr runs the stat prefilter, then a content hash, against the manifest for each touched path.
4. For each confirmed change, GraphMgr reparses and schema-validates the file and upserts, updates, or removes the corresponding index rows and embeddings.
5. API queries reflect the change.

## Extensions

### Extension 3a. A file is only half-written when the event fires

1. The debounce window re-checks the stat; if the file is still changing, the check is deferred to the end of the window.
2. If the content still fails to parse or validate, the entity is marked invalid per-file and everything else syncs normally.
3. The file is retried on its next detected change.

### Extension 4a. A git pull or merge brings many files in at once

1. The bulk event burst is absorbed by the debounce window.
2. The poll confirms the final state of every touched path; the index converges to the merged tree.
3. Return to step 5.

### Extension 5a. The developer deletes a node that still has edges

1. The node is removed from the index.
2. The edges whose src or dst no longer resolves are handled per the dangling-edge policy (open issue).

## Open Issues

- The policy for edges left dangling after an out-of-band node deletion (flag versus cascade) is not yet resolved; write-time referential integrity covers API writes only.
- The watcher library choice and the network-filesystem fallback behavior are implementation details, not yet fixed.

## Related Information

### Notes

- REQ 2fdb2f37-bba8-479a-9bbb-50a2127e7132: Tolerance of Out-of-Band Manual Edits and Git Operations
- REQ fd11706d-0152-4319-aa9c-d9bb6935abed: Hybrid Watch and Poll Triggers with Bounded Watch Count
- REQ d20e1a61-7dad-4978-812b-53f35287acf4: Per-File Parse and Validation Failure Isolation
- RSK 6342b261-1cd0-4cdb-8ceb-70c284b8e2dd: Graph Corruption from Out-of-Band Manual Edits
- DEC 3cccd4cb-f488-4a6f-8005-f38d2b5e92ef: Change Detection: Git-Independent Filesystem Manifest with Hybrid Watch and Poll

### Assumptions

- The repository working tree is the canonical location; the developer edits in place.
- Editors may save via rename, producing CREATE plus DELETE plus MODIFY for one logical save.
