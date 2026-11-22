---
classification: null
created: '2026-10-13T04:34:08.715+02:00'
id: 797b22c3-dd5d-4a66-ac74-64a1f1ce36ee
status: draft
type: qa
updated: '2026-10-13T04:34:08.715+02:00'
version: 1.0.0
---

# Node/Link Graph Architecture — Elicitation Q&A (2026-10-10/11)

## General

### Introduction

This Q&A record captures the requirements-elicitation interview that produced the initial architecture of biz.dfch.GraphMgr, the generic typed node/link graph system. It was conducted on 2026-10-10 and 2026-10-11 between the project owner (interviewee) and an AI assistant (interviewer), in session ses_ed8c0da78ffeLB78W5hsXJNOor. The full reconstructed transcripts are kept at the repository root as first-conversation.md and session-first-conversation-ses_ed8c.md. The distilled problem statement is PRB e1dd6083-cbd6-42da-a285-bf9e80c21d5a: Generic Typed Node/Link Graph with Git-Trackable Text Storage; the locked decisions are recorded in the four decision documents listed under More Information. Within each category, question and answer pairs appear directly one after another, with no heading of their own.

### Raw Requirements

The interviewee's opening message, preserved verbatim (including original spelling):

"I want to build a software that implements a node set with connectors between each other (a regular "network"). Each node can be of a "type" or "stereotype" with a fixed set of attributes that are common to each node (core attributes) and other attributes specific to the "type". in classic dbs this could be represented in a single table. The other "table" would keep track of the relations or "links" or "connectors" between nodes. These links can also be of different types, similar to UML connectors. They have a direction (src, dst).

The whole management of this node, link structure must be abstract by a ABI, REST API and MCP.

I want to find out, if there is a software library that implements this. If there is not good fit for this with an existing library, I want to know if it is feasible to implement this "text based". The reason: I want to be able to commit changes to the nodes and links via git, so I can track changes precisely. I think, with a normal DB I cannot do this, beause of the binary format. Access would always go through the API, so access and concurrency is controlled.

What are your thoughts on this?"

## Elicitation Context

> **0.0010**: What is the primary domain for this node/link network?

A generic, domain-agnostic graph model that can be extended; it must not be tied to one domain (for example software architecture, organizations, or processes).

> **0.0020**: How should links or connectors be represented?

Not as inline text references inside the node files. Links must be stored separately, as a dedicated "link table" of first-class edge entities, similar to UML connectors, with direction (src, dst) and their own type and attributes.

> **0.0030**: Are multi-hop graph queries needed, or mostly simple lookups?

Preferrably multi-hop. The idea floated: read the text files into an in-memory relational database and have change detection when the backing text files change, which would give a strong query language.

> **0.0040**: What should happen next?

Draft a concrete architecture plan.

> **0.0050**: How should this discussion be persisted?

Write the reconstructed conversation to the repository root as first-conversation.md; make no opencode or specmgr configuration changes in that session; no structured documents were created then.

## Functional Suitability

> **1.0010**: How should type schemas be defined and extended?

As data-driven JSON Schema files under schemas/node/ and schemas/edge/, validated generically. Adding a new node or edge type means adding a schema file, not shipping new code. Code-driven per-type models (the specmgr approach) were rejected because GraphMgr's types are open, not fixed in advance.

> **1.0020**: Can the type of an edge change after creation?

No. Edge type is immutable once created; a "retype" is always delete-old plus create-new, never an in-place type change.

> **1.0030**: Do non-type edge attributes get updated in place?

Yes — for example label or weight — and that is not a rare case.

> **1.0040**: What does the realistic edge usage pattern look like?

(1) create a node, (2) create edges to other nodes, (3) fix mistakes (delete and recreate edges), (4) much later: sparse lifecycle adds and occasional deletes.

> **1.0050**: When should a dangling edge endpoint be rejected?

At write time, by the API: an edge whose src or dst does not resolve to an existing node is rejected. This is possible precisely because edges are dedicated entities rather than inline tags.

> **1.0060**: Which serialization format should the entity files use (YAML, JSON, TOML, or other)?

TODO: answer pending

> **1.0070**: Which programming language and concrete tech stack will be used?

TODO: answer pending

## Performance Efficiency

> **2.0010**: How are 100k+ nodes handled as files instead of one flat directory?

By sharding on a prefix of the entity's own UUID — the same technique git uses for its object store. Random UUIDs distribute evenly, so prefix slicing needs no extra hashing.

> **2.0020**: Should each edge be a JSONL line in a large file per edge type, instead of one file per edge?

Analyzed and rejected as the default. JSONL update and delete cost is O(shard size) (lines are not fixed-width), writes to the same shard serialize, and change detection needs an extra line-level diff per shard. Because non-type edge attributes are updated in place and the workload is bursty, that cost would only worsen over the system's lifetime. One file per edge, directory-sharded exactly like nodes, was chosen; sharded JSONL is deferred as a possible opt-in for a specific, measured, high-volume, append-dominant, rarely-updated edge type.

> **2.0030**: Should the sharding directory be id[0:2] (256 shards) or id[0:3] (4096 shards)?

id[0:2], 256 shards per type. File-access performance does not meaningfully differ at any realistic scale; the deciding factor is the Linux inotify watch-descriptor budget, which scales with directory count, not file count. 4096 shards per type could approach or exceed default fs.inotify.max_user_watches without sysctl tuning.

> **2.0040**: What is the effect of many very small files on disk usage?

Dominant effect is filesystem block-size rounding (commonly 4 KB per file): 100k nodes plus 1M edges allocate roughly 4.3 GB even if the actual content totals tens of MB — a non-issue in absolute terms. The more likely real failure mode is inode exhaustion on filesystems with a fixed inode count; deployment must provision a generous inode budget or use a filesystem with dynamic inode allocation (XFS, APFS, NTFS).

> **2.0050**: Which embeddable SQL engine should back the in-memory query index?

SQLite (stdlib, ubiquitous, solid WITH RECURSIVE support for multi-hop traversal). DuckDB was considered and rejected in favor of SQLite's simplicity and ubiquity.

> **2.0060**: Should derived state (the change manifest, the SQLite index, the embeddings) persist across process restarts?

No. Always rebuild from scratch on every process start — one simple consistency invariant (process start means a guaranteed-correct full resync) — at the cost of a slower cold start as the corpus grows, which was accepted. This rule covers the manifest, the SQLite index, and the vector embeddings alike.

> **2.0070**: What is the long-term write-lock granularity for entity writes?

Type-level granularity is the starting point (simpler and safer). The one-file-per-entity layout would support finer per-id locking if ever needed; the long-term granularity is not yet fixed.

## Compatibility

> **3.0010**: Which OS-level file-watching mechanisms are available, and is the design portable across operating systems?

inotify on Linux, FSEvents on macOS, ReadDirectoryChangesW on Windows. A watchdog-style library abstracts all three over the same OS primitives, so the design is the same regardless of the chosen language. The binding constraint is Linux/inotify, which matters because the backend server is most likely deployed on Linux.

> **3.0020**: Do native file watches work reliably over network filesystems?

No — not on NFS or some SMB setups. There, the periodic poll becomes the primary detection mechanism rather than a backstop.

## Interaction Capability

> **4.0010**: Will individual edges be hand-edited directly and frequently?

Mostly via the API, but hand-editing is allowed and must not be dangerous. It does not need to be JSONL. This, together with the in-place attribute-update cost, drove the one-file-per-edge choice.

> **4.0020**: Must nodes be able to hold additional data files?

Yes. Nodes are represented as an id directory containing a convention-based index file (for example `index.<ext>` or a `node.*` glob), without changing the node/edge relationship. Sibling files are opaque payload: served on request, not parsed or indexed. Edges will not need this unless it later makes sense to unify.

> **4.0030**: What is the exact naming convention for the node index file?

TODO: answer pending

## Reliability

> **5.0010**: How are filesystem changes to the text files detected when they happen outside the API?

Through a filesystem-level manifest of the form { path -> (mtime, size, content_hash) }, completely independent of git: a real-time OS-level watchdog feed, debounced by roughly 100-300 ms, plus a periodic poll as a safety net. Verification is a cheap stat() pre-filter (mtime plus size) before paying for a full content hash (a fast non-cryptographic hash such as xxHash is sufficient; this is not a security use case).

> **5.0020**: Must detection work even when the changed files are not staged or committed?

Yes. The git-state-based approach (diffing against a tracked commit SHA) was rejected because it only works if changes are staged or committed. Git is demoted to a history/audit layer only; it plays no role in deciding whether the query index needs updating.

> **5.0030**: How should the sync check be scheduled?

Hybrid: a watchdog for real-time reaction, plus a periodic poll that sweeps the same stat-then-hash check across the tracked tree and catches anything the watcher missed (dropped events, watch-descriptor exhaustion, network filesystems).

> **5.0040**: How are partial writes and mid-save files handled?

By the debounce window (a write still in progress is detected by a stat that keeps changing) plus treating parse or schema-validation failure as a non-raising, per-file condition: one bad or mid-write file never blocks sync of everything else; it is marked invalid or stale and retried on the next detected change to that path.

> **5.0050**: How is the inotify watch count kept bounded given folder-per-node storage?

inotify does not report grandchild events, so watching every node directory would make the watch count proportional to total node count. The decision: inotify watches are held at the shard-directory level only (fast detection of new entities appearing), and the periodic poll is the authoritative mechanism for detecting content changes inside existing node folders — keeping the watch count bounded by shard count, not node count.

## Security

## Maintainability

> **7.0010**: Should all access surfaces share one implementation of the business logic?

Yes. One shared core library (the ABI) implements validation, locking, file I/O, git commit, index sync, and queries. REST (for example FastAPI) and MCP are thin, logic-free transport wrappers around that single core — never duplicating business logic per transport, the same discipline specmgr itself follows.

## Flexibility

> **8.0010**: Should git be the source of truth for change detection?

No. Detection is pure filesystem content tracking; the same mechanism would work pointed at a non-git directory. Git remains the history, rollback, and audit layer.

> **8.0020**: Is sharded JSONL still an option for the future?

Yes — as a targeted opt-in optimization for a specific edge type that is measured to be high-volume, append-dominant, and rarely updated. It is not the baseline; it would be adopted per edge type, not for all edges.

> **8.0030**: Can similarity or related-node search be added without changing the storage layout?

Yes, and it is additive. Embed each entity's name plus description with a small local model (fastembed / bge-small class, as precedent exists in specmgr's own find_related and find_similar_text tools), store the embeddings as another derived, rebuildable index via the sqlite-vec extension inside the same SQLite database (one engine, one lifecycle; an external vector-DB service was rejected as an inconsistent operational footprint; an embedded ANN library such as faiss or hnswlib is the fallback if sqlite-vec proves insufficient). The embedding sync piggybacks on the same change-detection pipeline, and embeddings are recomputed on every startup, consistent with the no-persisted-derived-state rule.

## Safety

## More Information

Source documents: first-conversation.md (curated record) and session-first-conversation-ses_ed8c.md (raw transcript), both at the repository root, from session ses_ed8c0da78ffeLB78W5hsXJNOor.

Artifacts created from this discussion:

- PRB e1dd6083-cbd6-42da-a285-bf9e80c21d5a: Generic Typed Node/Link Graph with Git-Trackable Text Storage
- GOL 86b457d9-58b0-4c9c-8c53-cf7849a72e75: Provide a Generic, Extensible Typed Node/Edge Graph Model
- GOL 1585066f-9fc7-44bb-a3be-3c5e6cf91cde: Keep Graph Data Precisely Trackable in Git
- GOL a72b270f-2caf-4654-bf72-9dfa750f26ea: Support Multi-Hop and Similarity Queries of the Graph
- GOL 0503a9d2-0a80-4369-a295-ed978e3e4bef: Detect and Absorb Changes Made Outside the API
- DEC 2c4a32c5-c9a7-4ad7-8740-53288b3ff15b: Overall Approach: Bespoke System Following the specmgr Pattern
- DEC a6235507-22fa-4aed-a45e-d9707c0f13bd: Storage Layout: One File per Entity, 256 Id-Prefix Shards
- DEC 3cccd4cb-f488-4a6f-8005-f38d2b5e92ef: Change Detection: Git-Independent Filesystem Manifest with Hybrid Watch and Poll
- DEC 4a3b078a-852c-4a4e-87b5-816e17dd0a44: Derived State and Query Layer: Full Rebuild on Startup, SQLite with sqlite-vec

The Security and Safety categories are intentionally empty: the discussion elicited no security- or safety-relevant requirements for this system.
