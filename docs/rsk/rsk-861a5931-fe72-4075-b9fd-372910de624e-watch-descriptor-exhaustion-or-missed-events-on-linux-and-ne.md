---
classification: null
created: '2026-10-13T04:03:33.268+02:00'
id: 861a5931-fe72-4075-b9fd-372910de624e
status: open
type: rsk
updated: '2026-10-13T04:03:33.268+02:00'
version: 1.0.0
---

# Watch-Descriptor Exhaustion or Missed Events on Linux and Network Filesystems

## Cause

On Linux, inotify watches are per-directory, not recursive: a recursive watcher holds one watch descriptor per subdirectory. The default fs.inotify.max_user_watches limit (commonly 8192-65536) can be approached by many shard directories times many entity types. On network filesystems (NFS, some SMB), native watch events do not propagate reliably at all.

## Trigger

Deployment with a large number of entity types and directories, or deployment on a network filesystem where watch events are dropped or never fire.

## Consequence

Real-time detection degrades: changes appear in the query index only on the poll interval, or the watcher silently misses events and the debounced pipeline reacts late.

## Scope

- Change-detection subsystem (watchdog and poll components)

## Initial Assessment

### Probability 2

### Impact 3

## Strategy

reduce

## Mitigation

Hold inotify watches at the shard-directory level only, so the watch count is bounded by shard count (256 per type) rather than entity count. Treat the periodic poll as the authoritative mechanism by design, so missed events self-heal within one poll interval. Document inotify limits and network-filesystem behavior in the deployment guide.

## Residual Assessment

### Probability 1

### Impact 2

## Owner

Development lead (to be assigned).

## Tags

- change-detection
- linux
- deployment

## Source

First design discussion, 2026-10-10/11 (first-conversation.md sections 5.3 and 5.5).

## More Information

See DEC 3cccd4cb-f488-4a6f-8005-f38d2b5e92ef: Change Detection: Git-Independent Filesystem Manifest with Hybrid Watch and Poll. macOS (FSEvents) and Windows (ReadDirectoryChangesW with subtree) do not have the per-directory limitation; it is specifically a Linux/inotify concern, which matters because the backend is most likely deployed on Linux.
