---
classification: null
created: '2026-10-13T04:13:54.698+02:00'
id: fd11706d-0152-4319-aa9c-d9bb6935abed
status: draft
type: req
updated: '2026-10-13T04:13:54.698+02:00'
version: 1.0.0
---

# Hybrid Watch and Poll Triggers with Bounded Watch Count

GraphMgr shall use OS-level file watches on shard directories only, together with a periodic full-tree poll as the authoritative mechanism for content changes inside existing files and node folders; the number of OS watches held shall be bounded by the number of shard directories, not by the number of entities.

## Description

On Linux, inotify watches are per-directory and a recursive watcher must hold one watch descriptor per subdirectory; watching every node folder would make the watch count proportional to total node count and could exceed default OS limits. Watch events are debounced by roughly 100-300 ms to absorb editor save-as-rename patterns and bulk merge churn. On network filesystems, where native watches do not fire reliably, the poll is the primary mechanism.

## Characteristics

1. Performance Efficiency
2. Reliability
3. Compatibility

## Level

MUST

## Priority

20

## Tags

- change-detection
- linux
- scale

## Source

First design discussion 2026-10-10/11 (first-conversation.md sections 4.3, 5.3, and 5.5); DEC 3cccd4cb-f488-4a6f-8005-f38d2b5e92ef.

## Related Artifacts

### Decisions

- DEC 3cccd4cb-f488-4a6f-8005-f38d2b5e92ef: Change Detection: Git-Independent Filesystem Manifest with Hybrid Watch and Poll
- DEC a6235507-22fa-4aed-a45e-d9707c0f13bd: Storage Layout: One File per Entity, 256 Id-Prefix Shards

### Risks

- RSK 861a5931-fe72-4075-b9fd-372910de624e: Watch-Descriptor Exhaustion or Missed Events on Linux and Network Filesystems
