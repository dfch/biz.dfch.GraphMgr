---
classification: null
created: '2026-10-13T03:59:10.706+02:00'
id: a6235507-22fa-4aed-a45e-d9707c0f13bd
status: accepted
type: dec
updated: '2026-10-13T04:00:48.749+02:00'
version: 1.0.0
---

# Storage Layout: One File per Entity, 256 Id-Prefix Shards

## Context and Problem Statement

The graph must scale to 100k+ nodes and a larger number of edges (edges outnumber nodes). The initial symmetric proposal put every entity of a type in one directory, which degrades git operations and filesystems at scale. Nodes must additionally hold arbitrary attachment files. The layout had to be settled before the change-detection and query designs, because file shape determines how changes are detected and how diffs look.

## Decision Drivers

- Atomic, localized git diffs: one change = one file, reviewable and blamable individually; minimal merge-conflict surface for concurrent writers.
- O(1) update and delete regardless of total entity count: non-type edge attributes (label, weight) are updated in place, and the real workload is bursty (create node, wire up many edges, fix mistakes via delete + recreate, then sparse lifecycle changes).
- Hand-edit safety: editing one entity must not risk corrupting unrelated records (edges are mostly API-managed, but manual editing must stay safe).
- Linux inotify watch budget: watches are per-directory; the backend is most likely deployed on Linux, so the number of watched directories matters, file count does not.
- Inode budget: many small files each consume an inode and at least one filesystem block; folder-per-node adds a directory inode per node.

## Considered Options

One file per entity (directory-sharded) versus sharded JSONL for edges (many records per shard file), and, within sharding, the prefix length (id[0:2] = 256 shard directories versus id[0:3] = 4096) and the shard key (the entity's own random id versus the src/dst node id). Nodes as single files versus nodes as folders with a convention-based index file plus opaque sibling attachments were also weighed.

## Decision Outcome

A symmetric one-file-per-entity design is chosen for both nodes and edges. Nodes live in a folder `nodes/<type>/<id[0:2]>/<id>/` containing a convention-named index file (the node's attributes) plus arbitrary sibling attachment files, which are opaque payload served on request, not parsed or indexed. Edges live as a single file `links/<type>/<id[0:2]>/<id>.<ext>`, with no attachments. Both are directory-sharded by the first two hex characters of the entity's own id (256 shards per type); edges are sharded by their own id, never by src or dst. Sharded JSONL is deferred entirely as a possible future opt-in optimization for a specific, measured high-volume, append-dominant, rarely-updated edge type; it is not the default. The serialization format itself is deliberately left open (YAML was only a working placeholder).

### Consequences

- One storage code path, one change-detection algorithm (stat then hash then reparse one file), and one locking model for both entity classes.
- O(1) update/delete and full per-entity write parallelism; a single node's burst of edges spreads evenly across shards because the shard key is the edge's own random id.
- Watch-descriptor count stays bounded by the shard count (256 per type) rather than the entity count, which keeps Linux inotify usage under default limits without sysctl tuning; per-node watches for attachment-aware content detection are rejected (see the change-detection decision).
- Many small files inflate raw disk usage through block-size rounding (roughly 4.3 GB allocated for 100k nodes + 1M edges at 4 KB blocks) and consume a large number of inodes; the more likely failure mode is inode exhaustion, which deployment must provision against.
- Adding a new type requires only a schema file plus the conventional directory; the index-file naming convention is still open.

### Confirmation

The user confirmed the symmetric one-file-per-entity model, the shard granularity id[0:2], and the folder-per-node attachment support during the 2026-10-10/11 design discussion after the write/update cost analysis of sharded JSONL (first-conversation.md sections 5.1-5.6).

## Roles and Responsibilities

### Accountable

The project owner (user of the 2026-10-10/11 design discussion).

### Responsible

- The GraphMgr development team (to be assigned at implementation start).

### Consulted

- The 2026-10-10/11 design discussion, including the edge mutability and churn analysis.

### Informed

- All future maintainers of the GraphMgr specification documents.

## Tags

- storage
- sharding
- layout

## Source

Session ses_ed8c0da78ffeLB78W5hsXJNOor, recorded in first-conversation.md section 5 and in session-first-conversation-ses_ed8c.md (both at the repo root).

## Related Artifacts

### Goals

- GOL 1585066f-9fc7-44bb-a3be-3c5e6cf91cde: Keep Graph Data Precisely Trackable in Git
- GOL 86b457d9-58b0-4c9c-8c53-cf7849a72e75: Provide a Generic, Extensible Typed Node/Edge Graph Model

## Pros and Cons

### Option 1: Symmetric one-file-per-entity, directory-sharded by own id (256 shards per type)

Pros: atomic localized git diffs; O(1) update/delete independent of total count; full per-entity write parallelism; trivial 1:1 change detection (file changed = entity changed); safe manual editing; watch count bounded by shard count. Cons: large file and inode counts at scale (block-rounding disk overhead, inode-exhaustion risk); more open/close syscalls during cold-start rebuild.

### Option 2: Sharded JSONL for edges (many records per shard file)

Pros: far fewer files; sequential bulk reads; cheap true appends. Cons: update and delete cost O(shard size) because JSONL lines are not fixed-width, and the cost only worsens over the system's lifetime as shards accumulate unrelated edges; writes to the same shard serialize (parallelism capped at shard count); change detection needs an extra line-level diff per shard; hand-editing one line among many is risky. Only wins if the edge workload is create-heavy with rare updates/deletes, which the confirmed workload (in-place attribute updates, bursty fix-mistakes cycles) is not.

## Updates

<!-- Newest entry first -- prepend new entries directly below this comment. -->

### 2026-10-13 03:52:00.000+02:00 - Created

Decision recorded from the 2026-10-10/11 design discussion; the storage model was locked after the edge mutability and shard-granularity analysis.
