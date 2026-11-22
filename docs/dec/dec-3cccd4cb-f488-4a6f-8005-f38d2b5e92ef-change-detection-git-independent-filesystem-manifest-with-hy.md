---
classification: null
created: '2026-10-13T03:59:48.668+02:00'
id: 3cccd4cb-f488-4a6f-8005-f38d2b5e92ef
status: accepted
type: dec
updated: '2026-10-13T04:00:50.804+02:00'
version: 1.0.0
---

# Change Detection: Git-Independent Filesystem Manifest with Hybrid Watch and Poll

## Context and Problem Statement

Files can be edited directly, outside the API (manual edits, git pull, merges), and the user explicitly required that changes be detected even when they are not staged or committed. An initial git-state-based design (track the last-synced commit SHA, diff against HEAD, detect rewrites via merge-base) was therefore rejected: it only works if changes are staged or committed. Detection must answer "does the query index need updating?" for any change that lands on disk, however it got there. Folder-per-node storage (`nodes/<type>/<shard>/<id>/index.<ext>`) additionally complicates OS-level watches, because inotify does not report grandchild events.

## Decision Drivers

- Must catch uncommitted working-tree edits (user requirement).
- Git is demoted to history/audit only; it plays no role in deciding whether the index needs updating.
- Linux inotify watches are per-directory and the backend is most likely deployed on Linux; watch count must stay bounded by shard count, not node count.
- Network filesystems do not propagate native watch events reliably, so a polling mechanism is mandatory, not optional.
- Editors often save via rename (CREATE + DELETE + MODIFY for one logical save), and bulk git merges fire many events at once; raw events need debouncing.
- A watch event can fire while a file is only half-written; one bad or mid-write file must never block synchronization of everything else.

## Considered Options

Git-state-based detection (last-synced SHA, git diff, merge-base ancestor checks) versus pure filesystem content tracking (in-memory manifest path to mtime/size/content-hash); committed-state-only synchronization versus live working-tree synchronization; and, for trigger scheduling, poll only, watchdog only, or a hybrid of both.

## Decision Outcome

Detection is pure filesystem content tracking, completely decoupled from git. An in-memory manifest of the form { path -> (mtime, size, content_hash) } is the single source for "did this change". Two triggers feed it in a hybrid: (1) a recursive OS-level watchdog, debounced by roughly 100-300 ms to absorb editor save-as-rename patterns and bulk merge churn; on Linux the inotify watches are held at the shard-directory level only, so the watch count is bounded by shard count; (2) a periodic poll that sweeps the same stat-then-hash check across the tracked tree and is the authoritative mechanism for detecting content changes inside existing files and node folders (and the primary mechanism on network filesystems). Verification is two-stage: a cheap stat() pre-filter (mtime + size) before paying for a full content hash, and a fast non-cryptographic hash (xxHash-class; this is not a security use case). A parse or schema-validation failure on one file is a non-raising, per-file condition: that entity is marked invalid/stale, everything else syncs normally, and the file is retried on its next detected change.

### Consequences

- Changes are detected the instant they hit disk, regardless of git add/commit/stage state; the same mechanism would work on a non-git directory.
- Content changes inside an existing node folder are reflected on the poll interval, not instantly (the deliberate trade for a bounded watch count).
- The manifest is held in memory only (see the derived-state decision); the full-tree walk at startup repopulates it.
- Debounce plus per-file failure isolation make partial writes and malformed hand edits recoverable without process restarts.

### Confirmation

The user rejected the git-state approach ("it only works if we stage/commit after each change") and the committed-only synchronization option, and confirmed the hybrid watchdog + periodic poll scheduling and the always-rebuild-from-scratch manifest policy during the 2026-10-10/11 design discussion (first-conversation.md sections 4.1-4.3 and 5.5).

## Roles and Responsibilities

### Accountable

The project owner (user of the 2026-10-10/11 design discussion).

### Responsible

- The GraphMgr development team (to be assigned at implementation start).

### Consulted

- The 2026-10-10/11 design discussion, including the four-scenario analysis (uncommitted edit, local commit, pull/merge, history rewrite).

### Informed

- All future maintainers of the GraphMgr specification documents.

## Tags

- change-detection
- filesystem
- reliability

## Source

Session ses_ed8c0da78ffeLB78W5hsXJNOor, recorded in first-conversation.md sections 4 and 5.5 and in session-first-conversation-ses_ed8c.md (both at the repo root).

## Related Artifacts

### Goals

- GOL 0503a9d2-0a80-4369-a295-ed978e3e4bef: Detect and Absorb Changes Made Outside the API
- GOL 1585066f-9fc7-44bb-a3be-3c5e6cf91cde: Keep Graph Data Precisely Trackable in Git

## Pros and Cons

### Option 1: Git-state-based detection (last-synced SHA, git diff, merge-base checks)

Pros: cheap precise deltas via git's object model; reuses git's blob hashes; index always reflects committed state. Cons: cannot see uncommitted working-tree edits, which the user explicitly requires; history rewrites force full rebuilds; couples the freshness of the query index to git plumbing the system should not depend on.

### Option 2: Pure filesystem content manifest, hybrid watchdog + periodic poll (chosen)

Pros: detects every change the moment it hits disk, staged or not; works on non-git directories; watch count bounded by shard count; poll makes dropped events and network-filesystem deployment safe; per-file failure isolation keeps one bad file from blocking the rest. Cons: recomputes hashes instead of reusing git's; must handle partial writes with debouncing; content changes inside existing node folders are detected on the poll interval rather than instantly.

## Updates

<!-- Newest entry first -- prepend new entries directly below this comment. -->

### 2026-10-13 03:52:00.000+02:00 - Created

Decision recorded from the 2026-10-10/11 design discussion; the git-independent manifest was adopted after the user's correction that detection must work without staging or committing.
