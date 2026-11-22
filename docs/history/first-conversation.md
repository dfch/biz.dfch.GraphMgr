# First Conversation: Node/Link Graph Architecture Discussion

Date: 2026-10-11

This document is a reconstructed record of the design discussion that shaped
the initial architecture for `biz.dfch.RelMgr` — a system for managing a
generic, typed node/edge network ("graph"), stored as git-trackable text
files, exposed through an ABI, a REST API, and an MCP server.

---

## 1. Initial problem statement

**User:** I want to build software that implements a node set with connectors
between each other (a regular "network"). Each node can be of a "type" or
"stereotype" with a fixed set of attributes common to each node (core
attributes) plus other attributes specific to the type — in a classic DB this
could be a single table. A second "table" tracks the relations/links/
connectors between nodes, also typed, similar to UML connectors, with
direction (src, dst). The whole management of this node/link structure must
be abstracted by an ABI, a REST API, and an MCP interface.

I want to find out if there is an existing software library that implements
this. If there's no good fit, I want to know if it's feasible to implement
this "text based" instead — because I want to be able to commit changes to
nodes and links via git, to track changes precisely, which I believe isn't
possible with a normal (binary) DB. Access would always go through the API,
so access and concurrency are controlled.

**Assistant — research findings:**

No mainstream library does exactly "typed node/edge graph, stored as plain
text, git-native, served over REST+MCP+a library API" as a single packaged
product, but the pattern is well proven. Survey of closest prior art:

- **Markdown + frontmatter + wikilinks (PKM tools)** — Obsidian, Foam,
  Dendron, Logseq, Quartz. Node = file, type = frontmatter field, link =
  wikilink/backlink index. Git-friendly, human-editable, but editor
  ecosystems, not an ABI/REST/MCP-governed system.
- **Architecture-as-code DSLs** — Structurizr DSL (C4 model): typed elements
  + typed, directed relationships, pure text, lives in git, has CLI/API.
  Strong fit if the domain were specifically software/system architecture.
- **"Git for data" databases** — TerminusDB (graph-native, schemas, REST,
  branch/diff/merge), Dolt (tabular, MySQL wire protocol, branch/diff/merge).
  Give git *semantics* with a real query engine, but data isn't stored as
  freely-editable text files on disk.
- **Text-based graph interchange formats** — DOT/Graphviz, GraphML, GEXF.
  Genuinely plain text and diffable, but no typed schema system, no API
  layer, no concurrency control.
- **Standards-based graph** — RDF/Turtle + SPARQL (rdflib, Oxigraph). Real
  typed graph model, text-serializable, but Turtle diffs are only clean with
  disciplined, stable serialization; steeper learning curve.
- **In-memory/embeddable graph engines** (as a *derived index*, not storage)
  — networkx, Kùzu. Text files are bad at multi-hop queries; the realistic
  pattern is text = source of truth, loaded into one of these as a
  rebuildable index for actual graph queries.

**Live proof-of-concept already available**: the `specmgr` MCP server
present in this environment implements almost exactly the requested pattern
for its own domain (requirements/specification documents): one markdown file
(or folder) per "node" (13 stereotypes: req, uc, tsk, qa, prb, gol, rsk, dec,
sop, feat, vcr, sysrs, adr), YAML frontmatter for core attributes (id, type,
status, created, updated, version), markdown body for type-specific
attributes (validated per type), and inline reference tags resolved into
directed links on read (`list_references`). All access goes through MCP
tools, with per-domain write locks and non-raising structured error results.
Referential integrity is soft (dangling refs are flagged, not blocked) — a
deliberate trade-off of the inline-tag design, later avoided in RelMgr's own
design by using dedicated edge entities instead.

**Verdict:** the text-based, git-tracked, API-gated approach is feasible and
proven. Key engineering concerns to design explicitly (not free from any
library): file layout, id scheme, link representation, referential
integrity policy, query needs beyond simple lookup, concurrency/locking,
per-type schema validation, and diff quality of the chosen serialization.

Three realistic paths were identified: (1) build bespoke following the
`specmgr` pattern, generalized; (2) RDF/Turtle + rdflib/Oxigraph for a
standards-based model; (3) Structurizr DSL if the domain were purely
software architecture. Given the stated requirement for a **generic,
extensible** graph model with **dedicated edge entities** (not inline refs),
path (1) was chosen.

---

## 2. Scope clarification

**User's answers to initial clarifying questions:**

- Primary use case: a **generic, domain-agnostic graph** model, extensible —
  not tied to one domain.
- Links must **not** be inline text references inside node files — they must
  be stored separately, as a dedicated "link table" (first-class edge
  entities).
- Query needs: **multi-hop** graph queries are desired (not just simple
  direct-neighbor lookups). Idea floated: read the text files into an
  in-memory relational DB with change detection when the backing text files
  change, to get a strong query language.
- Next step requested: draft a concrete architecture plan.

---

## 3. First architecture draft

### 3.1 Core data model

```yaml
# Node (core attributes, fixed across all types)
id: <uuid>
type: <stereotype, e.g. "system", "actor", "concept">
schema_version: 1.0.0
created: <timestamp>
updated: <timestamp>
# type-specific attributes below, validated against that type's schema
```

```yaml
# Edge / Link (core attributes, fixed across all types) — first-class entity
id: <uuid>
type: <stereotype, e.g. "depends_on", "contains", "association">
src: <node id>
dst: <node id>
directed: true
schema_version: 1.0.0
created: <timestamp>
updated: <timestamp>
# type-specific attributes below (label, multiplicity, navigability, weight, ...)
```

Dedicated edge entities (vs. inline refs) let the API enforce **referential
integrity at write time** — rejecting an edge whose `src`/`dst` doesn't
resolve to an existing node — which an inline-tag design cannot do cleanly.

### 3.2 On-disk layout (initial symmetric proposal)

```
repo/
  schemas/
    node/<type>.schema.json
    edge/<type>.schema.json
  nodes/
    <type>/<id>.yaml
  links/
    <type>/<id>.yaml
```

One file per entity (node or edge) — atomic, localized git diffs; minimizes
merge-conflict surface vs. a shared growing list-file per type.

**Schema approach decided:** data-driven JSON Schema files under
`schemas/node/*.json` and `schemas/edge/*.json`, validated generically via
`jsonschema` — adding a new type means adding a schema file, not shipping
new code.

### 3.3 In-memory relational query index

The user's idea (text files → in-memory relational DB with change detection)
was adopted as the query layer:

- **Engine chosen: SQLite** (stdlib, solid `WITH RECURSIVE` support for
  multi-hop traversal; DuckDB was considered and rejected in favor of
  SQLite's simplicity/ubiquity).
- Tables: `nodes(id, type, created, updated, attrs JSON)`,
  `links(id, type, src, dst, directed, created, updated, attrs JSON)`.
- The index is a **disposable, rebuildable cache**, never the source of
  truth — the git working tree of text files always is.
- Harder graph algorithms (weighted shortest path, centrality) layer
  `networkx` on top of a materialized subgraph pulled via SQL, rather than
  forcing everything into recursive CTEs.

### 3.4 Concurrency & API layering

- In-process write lock (type-level granularity to start).
- One git commit per API-driven write.
- One shared core library (the ABI) implements validation, locking, file
  I/O, git commit, index sync, and queries. REST (e.g. FastAPI) and MCP are
  thin, logic-free transport wrappers around that single core — never
  duplicating business logic per transport (same discipline `specmgr`
  itself follows).

### 3.5 Decisions locked in at this stage

- Schema approach: **data-driven JSON Schema files** (not code-driven
  Pydantic-per-type).
- Git write path: **direct manual editing + git also allowed** outside the
  API (not solely API-mediated commits) — this later became the key
  constraint shaping the change-detection design (see §4).
- Query engine: **SQLite**.
- Next step: keep discussing/refining before writing anything down formally.

---

## 4. Change detection: filesystem edits outside the API

**Problem:** if files can be edited directly (not only through the API), how
does the system detect changes made outside its own write path?

### 4.1 First pass — git-state-based detection

Initial proposal distinguished four scenarios: (1) uncommitted edit, (2)
local commit, (3) pull/merge from a remote, (4) history rewrite (rebase,
force-push). Proposed using `git diff --name-status <old_sha> <new_sha>`
against a tracked "last-synced commit SHA", with `git merge-base
--is-ancestor` to detect rewritten history requiring a full rebuild, plus an
OS-level filesystem watch (`inotify`/`watchdog` on Linux, FSEvents on macOS,
`ReadDirectoryChangesW` on Windows) as a low-latency trigger, backed by a
periodic poll as a safety net (works over network filesystems where native
watches don't propagate reliably).

A policy choice was raised: should the live index reflect only
git-committed state, or also uncommitted working-tree edits?

### 4.2 Correction — must detect changes even without staging/committing

**User:** the git-diff approach only works if changes are staged/committed.
A mechanism is needed that detects changes on files even when they are
**not** staged or committed at all.

**Resolution:** detection must be **pure filesystem content-tracking,
completely decoupled from git**. Git is demoted to a history/audit layer
only; it plays no role in deciding whether the query index needs updating.

### 4.3 Final change-detection design

A filesystem-level manifest, independent of git:

```
{ path -> (mtime, size, content_hash) }
```

- **Watchdog** (real-time push): recursive OS-level watch on the data
  directories; events are debounced (~100–300ms) to absorb editor
  save-as-rename patterns and bulk git-merge churn.
- **Verify step**: cheap `stat()` (mtime+size) pre-filter before paying for
  a full content hash (a fast non-cryptographic hash, e.g. xxHash, is
  sufficient — this isn't a security use case).
- **Periodic poll** (safety net): sweeps the same stat-then-hash check
  across the tracked tree, catching anything the watcher missed (dropped
  events, watch-descriptor exhaustion, network filesystems where native
  watches don't fire at all).
- **Partial-write races**: handled via the debounce window plus treating
  parse/validate failure as a **non-raising, per-file condition** — one
  bad/mid-write file never blocks sync of everything else; it's retried on
  the next detected change to that path.
- **Manifest persistence across restarts: rejected.** Decision: **always
  rebuild from scratch on every process start** — simpler single
  consistency invariant (process start = guaranteed-correct full resync),
  at the cost of slower cold start at large scale. This same "rebuild
  everything, persist nothing" rule was later extended to the SQLite index
  and to the vector-embedding index (§6.5).

---

## 5. Storage-layout scaling questions

### 5.1 Edges: JSONL vs. one-file-per-edge (first pass)

**User:** since edges will outnumber nodes, would it be easier to store each
edge as a JSONL line in a large file per edge type, to avoid opening huge
numbers of files?

**Analysis:**

| | One file per edge | Sharded JSONL |
|---|---|---|
| Create (append) | O(1) | O(1)-ish (true append is cheap) |
| Update | O(1), independent of other edges | O(shard size) — JSONL lines aren't fixed-width, so replacing a line means rewriting everything after it |
| Delete | O(1) | O(shard size), same rewrite problem |
| Concurrent writers | Full per-edge parallelism | Capped at shard count — writes to the same shard serialize |
| Cold-start rebuild | More open/close syscalls | Fewer file handles, better sequential I/O |
| Change detection | Trivial 1:1 (file changed = entity changed) | Needs an added line-level diff per shard to find which record(s) changed |
| Manual hand-editing | Natural, one edge per file | Riskier — editing one line among many |

Initial recommendation: **sharded JSONL** by edge id prefix, given edges
expected to vastly outnumber nodes and (at that point) assumed to be
mostly append-driven. The user pushed back, favoring one-file-per-edge
(sharded like nodes) once hand-editing turned out not to be a primary
concern ("Mostly via API, but it does not need to be JSONL").

### 5.2 Deciding factor: edge mutability and churn

Follow-up questions established the real workload:

- Edge `type` is **immutable** once created — a "retype" is always
  delete-old + create-new, never an in-place type change.
- **Other edge attributes (e.g. label, weight) are updated in place** —
  not a rare case.
- Real usage pattern: *(1) create a node → (2) create edges to other nodes →
  (3) fix mistakes (delete and recreate edges) → (4) much later: sparse
  lifecycle adds and occasional deletes.*

Given in-place attribute updates are a real, recurring cost, and the
workload is bursty (many creates/deletes/updates clustered around a single
node's creation), **sharded JSONL's O(shard-size) update/delete cost would
only get worse over the system's lifetime** as each shard accumulates more
unrelated edges. One-file-per-edge avoids this entirely: O(1) update/delete
regardless of total edge count or shard age.

**A key related design point**: edge shard keys must be derived from the
**edge's own random id**, not from `src`/`dst` node id — sharding by `src`
would cluster every edge created during one "wire up a new node" burst into
a single shard, the worst case for contention exactly during that workflow.

**Final decision:** symmetric design — **one file per entity for both
nodes and edges**, directory-sharded by the entity's own id prefix. Sharded
JSONL deferred entirely as a possible future opt-in optimization for a
specific, measured high-volume + append-dominant + rarely-updated edge type
— not the default.

### 5.3 Shard granularity: `id[0:2]` vs. `id[0:3]`

**User:** considered `id[0:3]` (4096 shard dirs). What's the performance and
filesystem-watcher impact?

- **File-access performance**: no meaningful difference between 256 and
  4096 shards at any realistic scale — modern filesystems (ext4 htree, XFS
  B+tree dirs, APFS, NTFS) handle thousands of files per directory without
  issue; 256 shards is already generous headroom.
- **Filesystem-watcher cost — the actual deciding factor, and OS-specific**:
  on Linux, `inotify` watches are per-directory, not recursive natively — a
  recursive watcher must hold one watch descriptor per subdirectory. 4096
  shards × many types could approach or exceed typical
  `fs.inotify.max_user_watches` defaults (commonly 8192–65536), requiring a
  sysctl change in deployment. macOS (FSEvents) and Windows
  (`ReadDirectoryChangesW` with subtree=true) don't have this per-directory
  limitation — it's specifically a Linux/inotify concern, which matters
  since a backend server is most likely deployed on Linux.
- Watch count scales with **directory count**, not file count.

**Decision: `id[0:2]` — 256 shards per type.** No performance benefit from
going finer, and it keeps total watched-directory count comfortably under
default OS limits without needing sysctl tuning.

### 5.4 Serialization format

**User note:** YAML was only ever a working placeholder, not a committed
decision. All discussion (sharding, change detection, locking) is format-
agnostic and applies equally to YAML/JSON/TOML or any structured text
serialization. Left open.

### 5.5 Nodes as folders (attachment support)

**User:** nodes must be able to hold additional data files, so they should
be represented as an `<id>/` directory with a convention-based index file
(e.g. `index.yaml` or `node.*`), without changing the node/edge
relationship. Edges will not need this unless it later makes sense to
unify.

**Resulting layout:**

```
nodes/<type>/<id[0:2]>/<id>/index.<ext>   (+ arbitrary sibling attachment files)
links/<type>/<id[0:2]>/<id>.<ext>          (single file, no attachments)
```

Precedent: `specmgr`'s own `feat` domain already uses folder-per-entity for
exactly this reason (it's the one domain needing multiple files).

**Consequence for change detection**: only the convention-named index file
should feed the SQL index; sibling attachments are opaque payload (served
on request, not parsed/indexed). More importantly, this reopens the watch
cost problem *worse* than §5.3: to get real-time `inotify` events on
`<id>/index.yaml` specifically requires a watch on **every individual
node's own directory** (inotify doesn't report grandchild events), i.e. one
watch per node, not per shard — a much bigger number, proportional to total
node count.

**Decision:** `inotify` stays at the **shard-directory level only** (fast
detection of new node folders appearing); the **periodic poll** becomes the
authoritative mechanism for detecting content changes *inside* existing
node folders — keeping watch count bounded by shard count, not node count.

### 5.6 Disk usage of many small files

**User:** with this sharding we'll have many very small files — what's the
effect on disk usage?

- Dominant effect: **filesystem block-size rounding**. Most filesystems
  (ext4, XFS, NTFS, APFS) allocate in fixed blocks (commonly 4KB); any file
  consumes at least one full block regardless of actual content size.
- Rough math: 100k nodes + 1M edges, each ≥1 block ≈ (100,000 + 1,000,000) ×
  4KB ≈ **4.3GB of raw allocated space**, even if actual content totals only
  tens of MB. Folder-per-node adds a second inode (the directory itself) on
  top of this.
- Two distinct costs: **raw disk space** (cheap in absolute terms on modern
  storage — a non-issue) vs. **inode exhaustion** (the more likely real
  failure mode — some filesystems, e.g. ext4 formatted with a conservative
  inode ratio, have a *fixed* inode count set at format time; you can run
  out of inodes while `df` still shows free space). Mitigation: provision
  the filesystem with a generous inode budget, or use one with dynamic
  inode allocation (XFS, APFS, NTFS).
- Git's own object store is unaffected (compresses/deduplicates internally)
  — the block-rounding cost only hits the raw working-tree checkout that
  the application and watcher actually interact with.

### 5.7 Similarity / related-node search (vector DB)

**User:** must be able to find "similar"/"related" nodes and edges based on
their name and description/content (defined in the index file). Can a
vector DB be used? Needed to know before settling the storage layout.

**Answer: yes — and it's additive, doesn't change or block the storage
layout.** Directly precedented by `specmgr`'s own `find_related`/
`find_similar_text` tools: local sentence embeddings via `fastembed`/
`bge-small`, cosine similarity, fully self-hosted (no external API calls).

Design:

1. Embed each node/edge's name + description text with a small local model.
2. Store embeddings as **another derived, rebuildable index**, same
   philosophy as the SQLite relational index. Chosen approach: the
   **`sqlite-vec`** extension, keeping embeddings in the *same* SQLite
   database as the relational data (one engine, one lifecycle) rather than
   standing up a separate vector-DB service (Qdrant/Weaviate/Milvus would
   be a heavier operational footprint, inconsistent with the all-embedded
   design built up so far). A dedicated embedded ANN library (`faiss`,
   `hnswlib`) was noted as a fallback if `sqlite-vec` proves insufficient.
3. **Sync lifecycle**: piggybacks on the same change-detection pipeline —
   when an index file's content changes, recompute its embedding and
   upsert it alongside the relational upsert.
4. **Query**: embed query text (or reuse a stored embedding) → nearest-
   neighbor search → top-K results, optionally joined with normal SQL/graph
   filters (e.g. "similar nodes, but only of type X").
5. It consumes only fields already planned to exist (name/description) — no
   change to the node/edge file format is required.

**Tension flagged and resolved:** embedding generation is CPU-bound and
non-trivial per item — re-embedding everything on every process restart
could be the first genuinely slow part of cold start at scale, unlike the
sub-millisecond stat/hash checks used for the rest of the sync. Asked
whether to special-case a persistent embedding cache (keyed by content
hash) as an exception to the "rebuild everything from scratch" rule.

**Decision: recompute all embeddings on every startup** — no persisted
cache, keeping full consistency with the general "no persisted derived
state" principle, accepting slower cold start as the corpus grows.

---

## 6. Consolidated architecture (as agreed)

**Data model**
- Generic, extensible typed node/edge graph. Core attributes fixed per
  entity class (id, type, created, updated, schema_version); type-specific
  attributes defined by a data-driven JSON Schema per type.
- Edges are first-class, separately-stored entities (`id`, `type`, `src`,
  `dst`, `directed`, attrs) — not inline references. Referential integrity
  enforced at write time.
- Edge `type` is immutable post-creation (retype = delete + create); other
  edge attributes can be updated in place.

**Storage**
- Nodes: `nodes/<type>/<id[0:2]>/<id>/index.<ext>` — folder-per-node,
  allows arbitrary sibling attachment files.
- Edges: `links/<type>/<id[0:2]>/<id>.<ext>` — single file per edge, no
  attachments, sharded by the edge's own id (not `src`/`dst`).
- 256 shards/type (`id[0:2]`) for both — chosen over 4096 because file-
  access performance is a non-issue either way, but watch-descriptor
  budget (Linux `inotify`) scales with directory count.
- File format itself: still open (YAML was a placeholder; discussion is
  format-agnostic).

**Change detection** (fully git-independent — must catch uncommitted edits)
- Filesystem-level content manifest (mtime/size prefilter → content hash on
  suspected change), not git plumbing.
- Hybrid trigger: `inotify`/watchdog at the shard-directory level only
  (catches new node/edge entities appearing quickly) + periodic poll as the
  authoritative mechanism for content changes inside existing files/node-
  folders (keeps watch count bounded by shard count, not node count).
- No persistence of the manifest, the SQLite index, or embeddings across
  restarts — everything rebuilds from scratch on every boot.
- Partial-write races handled via debounce + non-raising per-file parse/
  validate failure isolation.

**Query layer**
- SQLite: `nodes`/`links` tables, core columns + JSON attrs column,
  `WITH RECURSIVE` for multi-hop traversal. Purely a derived, rebuildable
  cache — the git working tree is the only source of truth.
- Vector similarity ("related node" search): local embeddings (`fastembed`/
  `bge-small`-style, precedented by `specmgr`), stored via the `sqlite-vec`
  extension to stay in one engine, synced through the same change-detection
  pipeline, fully recomputed on every startup.

**Concurrency & API**
- In-process write lock (type-level granularity as the starting point).
- One shared core library (ABI) implementing validation/locking/storage/
  index/query; REST and MCP are thin, logic-free transport wrappers around
  it.
- Git commit happens on every API-driven write (for audit trail); direct
  manual edits/independent git operations outside the API are also fully
  supported — which is exactly why change detection had to be filesystem-
  level rather than git-state-based.

**Disk/ops considerations flagged**
- Block-size rounding inflates raw disk usage for many tiny files (bounded,
  generally cheap at realistic scale) — not a real concern.
- Inode exhaustion is the more likely failure mode — filesystem must be
  provisioned (or chosen) to handle large file/directory counts.

**Still open / not yet decided**
- Serialization format (YAML/JSON/TOML/other).
- Programming language and concrete tech stack (explicitly not chosen yet).
- Index-file naming convention (e.g. `index.yaml` vs. a `node.*` glob).
- Exact edge-lock granularity long-term (started at type-level; one-file-
  per-edge would support finer per-id locking if ever needed).
- Implementation details of the watch/poll hybrid (library choice,
  network-filesystem fallback behavior).

---

## 7. Persisting this discussion

**User:** created `~/src/biz.dfch.RelMgr` to persist this discussion.
Initial request was to create a `specmgr` QA document and a DEC document for
the discussion, plus keep the full conversation history (not only a
summarized QA+DEC).

**Problem found:** in this session, the `specmgr` MCP server's directories
all resolve under `/tmp` (confirmed via the `specmgr://config` resource —
every domain's `env_var_set` is `false`, so each falls back to its built-in
default), not `~/src/biz.dfch.RelMgr`. The server is launched via the
`specmgr` entry in the opencode config (`~/docker/opencode/.config/opencode/
opencode.json`), which currently sets no `SPECMGR_DOCS_DIR` / `SPECMGR_ADR_DIR`
/ `SPECMGR_FEAT_DIR` environment variables at all.

A config-change plan (project-scoped `opencode.json` override, with the
required restart of opencode to pick up new environment variables) was
proposed, but the user explicitly declined any opencode/specmgr config
changes and simplified the request down to: **just write this conversation
to the root of `~/src/biz.dfch.RelMgr` as `first-conversation.md`.** No QA
document and no DEC document were created, per the final instruction.

---

*End of reconstructed transcript.*
