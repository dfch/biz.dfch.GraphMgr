---
classification: null
created: '2026-10-13T04:34:08.715+02:00'
id: 797b22c3-dd5d-4a66-ac74-64a1f1ce36ee
status: draft
type: qa
updated: '2026-11-22T12:33:30.298+01:00'
version: 1.0.0
---

# Node/Link Graph Architecture — Elicitation Q&A (2026-10-10/11)

## General

### Introduction

This Q&A record captures the requirements-elicitation interview that produced the initial architecture of biz.dfch.GraphMgr, the generic typed node/link graph system. It was conducted on 2026-10-10 and 2026-10-11 between the project owner (interviewee) and an AI assistant (interviewer), in session ses_ed8c0da78ffeLB78W5hsXJNOor. The reconstructed record first-conversation.md is kept in docs/history/; the raw transcript session-first-conversation-ses_ed8c.md is kept only locally in the repository root and is not committed. The distilled problem statement is PRB e1dd6083-cbd6-42da-a285-bf9e80c21d5a: Generic Typed Node/Link Graph with Git-Trackable Text Storage; the locked decisions are recorded in the four decision documents listed under More Information. Within each category, question and answer pairs appear directly one after another, with no heading of their own.

Questions added on 2026-11-22 (after a review of the specification set) carry a Context paragraph, the main options, and the reviewer's recommendation. The recommendations are suggestions to make answering easier, not decisions.

### Raw Requirements

The interviewee's opening message, preserved verbatim (including original spelling):

"I want to build a software that implements a node set with connectors between each other (a regular \"network\"). Each node can be of a \"type\" or \"stereotype\" with a fixed set of attributes that are common to each node (core attributes) and other attributes specific to the \"type\". in classic dbs this could be represented in a single table. The other \"table\" would keep track of the relations or \"links\" or \"connectors\" between nodes. These links can also be of different types, similar to UML connectors. They have a direction (src, dst).

The whole management of this node, link structure must be abstract by a ABI, REST API and MCP.

I want to find out, if there is a software library that implements this. If there is not good fit for this with an existing library, I want to know if it is feasible to implement this \"text based\". The reason: I want to be able to commit changes to the nodes and links via git, so I can track changes precisely. I think, with a normal DB I cannot do this, beause of the binary format. Access would always go through the API, so access and concurrency is controlled.

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

> **0.0060**: Who or what will use GraphMgr day to day (people through a CLI or UI, AI agents through MCP, other systems through REST), and on which machines?
>
> *Context:* The specs name three access surfaces but no concrete consumers. The answer drives several later answers: whether authentication is needed (6.0010), whether there is one shared server or one local instance per user (5.0060), and what shape query results need (4.0040).
>
> *Options:* (a) one user on a local machine, mostly agents through MCP; (b) a small team sharing one server; (c) a service that other systems call.
>
> *Recommendation:* Name the first real consumer and design the first release for it.

TODO: answer pending

> **0.0070**: Which features are in scope for the first release, and which can come later?
>
> *Context:* The accepted design contains the core library, the sharded file store, one git commit per write, the SQLite index with multi-hop traversal, watcher plus poll change detection, local embeddings with sqlite-vec, REST, and MCP. Building all of it before anything is measured carries a lot of risk. A smaller first release makes it possible to measure commit cost, poll time and cold start (2.0090 to 2.0120) before more decisions are fixed.
>
> *Options:* (a) everything at once; (b) core library, file store, git commits, SQLite index and poll-only change detection, plus one transport (MCP or REST); add the watcher, embeddings and the second transport later.
>
> *Recommendation:* (b).

TODO: answer pending

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
>
> *Context:* The format affects diff quality, hand-editing and parse time during the full rebuild at startup. YAML is readable, allows comments and multi-line text, but parses more slowly and has pitfalls (implicit typing, for example `no` read as false) unless a strict loader is used. JSON is fast, strict and maps one to one to JSON Schema, but has no comments and gives noisy diffs for long text. TOML is readable but awkward for nested data. Markdown with YAML frontmatter (the specmgr pattern) suits long descriptions. Whatever the format, the writer should emit a stable key order so that diffs stay small.
>
> *Recommendation:* YAML with a strict loader and a fixed key order; JSON if hand-editing turns out to be rare. Affects DEC a6235507-22fa-4aed-a45e-d9707c0f13bd (left open there) and UC 98135519-b97e-4df0-8386-45ee5a2fb66d.

TODO: answer pending

> **1.0070**: Which programming language and concrete tech stack will be used?
>
> *Context:* The repository is already set up as a Python 3.13 package with uv, and AGENTS.md assumes Python, but no decision records this. Python covers the planned parts well: sqlite3 (standard library), jsonschema, watchdog, fastembed, sqlite-vec bindings, FastAPI and the MCP Python SDK. Still open inside that choice is the git access method (git command line, pygit2/libgit2, or dulwich), which matters for commit cost (2.0120).
>
> *Recommendation:* Confirm Python and record it as a decision; choose the git library after a commit benchmark.

TODO: answer pending

> **1.0080**: What happens when a node that still has edges is deleted through the API?
>
> *Context:* The specs reject a new edge with a missing endpoint (REQ c7f59a15-31f8-4f10-adc4-becd73deb675) but say nothing about deleting a node. Without a rule, an API delete creates exactly the dangling edges the system is meant to prevent.
>
> *Options:* (a) reject the delete while edges exist, so the caller deletes the edges first; (b) cascade: delete the node and all its edges in one write and one commit; (c) allow the delete and mark the edges dangling. A cascade can touch many files, which ties in with batch writes (1.0110).
>
> *Recommendation:* (a) by default, with an explicit cascade option.

TODO: answer pending

> **1.0090**: How are dangling edges handled when they arise outside the API, for example when a node folder is deleted by hand, or an edge file that points to a missing node arrives by hand or through git pull?
>
> *Context:* Listed as open in UC 0b1ae5ce-bc8e-4e11-beb8-3b75455dfec8 and RSK 6342b261-1cd0-4cdb-8ceb-70c284b8e2dd ("flag versus cascade"). Changes made outside the API cannot be rejected, only reacted to.
>
> *Options:* (a) keep the edge file, mark the edge dangling in the index, leave it out of traversals, and report it; (b) delete the edge files automatically, which means GraphMgr changes (and maybe commits) files the user did not ask it to touch.
>
> *Recommendation:* (a). GraphMgr should not change files it did not write.

TODO: answer pending

> **1.0100**: Are `name` and `description` core attributes of every node and edge, or does each type's schema declare which fields are embedded for similarity search?
>
> *Context:* Similarity search (REQ 64d52fab-3260-4d96-aefe-a2e2aa4ad11f, UC 044b6e3d-fed2-4eaf-98c2-949ba2e6a0f7) works on name and description, but the core attribute lists (REQ 7eeb13d8-c1bd-45b8-8b24-bca94c2998f9, REQ 3d4b0381-c825-474b-acaf-3ada75a0ee56) do not contain them; the use case only assumes they exist.
>
> *Options:* (a) add `name` (required) and `description` (optional) to the core set of nodes and edges; (b) each schema marks the text fields to embed, for example with a custom `x-embed` keyword.
>
> *Recommendation:* (a) for `name`, which is also useful for display and lookup, plus (b) as an optional extension.

TODO: answer pending

> **1.0110**: Should the API offer a batch write, for example "create a node plus its edges", that is validated as one unit and produces one git commit?
>
> *Context:* REQ d851888c-2f99-4f84-b0b9-f9de01b55c9b requires one commit per API write. The main workflow (UC 98135519-b97e-4df0-8386-45ee5a2fb66d) is one node followed by many edges: N+1 separate writes and N+1 commits, and if one edge fails the earlier ones are already committed. A batch would also make a cascade delete (1.0080) a single commit.
>
> *Options:* (a) single-entity writes only; (b) add an all-or-nothing batch operation with one commit, and define "write" in REQ d851888c-2f99-4f84-b0b9-f9de01b55c9b as "one API call".
>
> *Recommendation:* (b).

TODO: answer pending

> **1.0120**: The entity type appears both in the path (`nodes/<type>/...`) and inside the file. Which one wins if they disagree, and what happens when a file is moved to another type folder by hand?
>
> *Context:* This can only happen through changes outside the API. A hand edit can also change an edge's `type` field, which REQ 427c23c8-e353-43f2-bb06-b361de9e1e1d says must never happen in place.
>
> *Options:* (a) a mismatch makes the entity invalid, handled by per-file isolation (REQ d20e1a61-7dad-4978-812b-53f35287acf4); (b) the path wins; (c) remove the type from the file and keep it only in the path.
>
> *Recommendation:* (a), and treat a changed edge type as invalid too.

TODO: answer pending

> **1.0130**: Should edges be included in similarity search, or only nodes?
>
> *Context:* REQ 64d52fab-3260-4d96-aefe-a2e2aa4ad11f says nodes and edges; the title of UC 044b6e3d-fed2-4eaf-98c2-949ba2e6a0f7 says nodes. Edges are expected to outnumber nodes about ten to one (1M versus 100k), so embedding edges multiplies embedding time and memory by about ten (see 2.0100). Many edges carry only a type and perhaps a label, which gives little to compare.
>
> *Recommendation:* Nodes only by default; an edge type can opt in through its schema.

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

> **2.0080**: What is the target size of the graph, for the first release and in the long term (number of nodes, edges and entity types, and the size of a typical entity file)?
>
> *Context:* The specs use "100k+ nodes" and "1M edges" as examples, but no requirement fixes a number. Several costs grow with it: commit time (2.0120), poll time (2.0110), cold start (2.0090) and inode use (RSK 380eb6dc-eb38-4046-8884-14b4058d6410). If the first release is closer to 10k entities, several scale concerns can wait.
>
> *Recommendation:* Give one number for the first release and one for the long term, and record them in a requirement.

TODO: answer pending

> **2.0090**: What is the longest acceptable time from process start until queries work, at the target size, and what should the system do while it is still rebuilding?
>
> *Context:* REQ 06f32855-c050-4fcd-8538-a3550ccff070 rebuilds all derived state at every start; RSK 8401d706-a3dd-4205-9675-f73daec6d144 names the risk but sets no limit. The use cases say "not ready" during the rebuild, but nothing says whether writes are accepted meanwhile.
>
> *Options:* (a) all queries report "not ready" until everything is rebuilt; (b) graph queries work as soon as the SQLite index is built, and only similarity search reports "not ready" until the embeddings are done. For writes during the rebuild: accept them or reject them.
>
> *Recommendation:* Set a number (for example "graph queries ready within N minutes at the target size") and choose (b).

TODO: answer pending

> **2.0100**: Should embeddings be kept between restarts in a cache keyed by a hash of model id, model version and the embedded text? This would revisit 2.0060 for embeddings only.
>
> *Context:* 2.0060 rejected all persisted derived state to avoid stale data. A content-keyed cache cannot go stale: changed text or a changed model gives a new key, so an old vector is never used for new text, and deleting the cache is always safe. The manifest and the SQLite index would still be rebuilt at every start. Rough, unmeasured estimate: a small local model on CPU embeds a few hundred to about a thousand short texts per second, so 100k nodes take about 2 to 15 minutes and 1.1M entities 20 minutes to over an hour, at every restart. A 384-dimension float32 vector needs about 1.5 KB, about 1.7 GB for 1.1M entities.
>
> *Options:* (a) keep 2.0060 as it is (no cache); (b) add the cache now; (c) keep (a) for the first release and add the cache if the benchmark misses the limit set in 2.0090.
>
> *Recommendation:* (c). Affects DEC 4a3b078a-852c-4a4e-87b5-816e17dd0a44, REQ 06f32855-c050-4fcd-8538-a3550ccff070 and RSK 8401d706-a3dd-4205-9675-f73daec6d144.

TODO: answer pending

> **2.0110**: How quickly must a manual edit to an existing node show up in queries? Should the node's main file move up into the shard folder so that file watches see the edit?
>
> *Context:* inotify watches are held only on shard folders (5.0050). An edit to `nodes/<type>/<shard>/<id>/index.<ext>` is one level deeper, so only the periodic poll sees it, and the poll must check every file (1.1M at the long-term example size; estimated a few seconds per run on a local disk, much longer on network filesystems). The delay for hand edits to nodes is therefore the poll interval. Alternative layout: `nodes/<type>/<shard>/<id>.<ext>` holds the node's data and `nodes/<type>/<shard>/<id>/` holds attachments only. The shard watch then sees node edits at once, and the poll is a safety net again, exactly as for edges.
>
> *Recommendation:* The alternative layout; it also answers 4.0030. Affects DEC a6235507-22fa-4aed-a45e-d9707c0f13bd, DEC 3cccd4cb-f488-4a6f-8005-f38d2b5e92ef and REQ 41498f3b-3061-43d1-b910-44cc5d859409.

TODO: answer pending

> **2.0120**: What is the acceptable time for one API write, including its git commit?
>
> *Context:* A normal `git add` plus `git commit` rewrites git's index file, which lists every tracked file. With about 1.1M files that file is on the order of 100 MB or more, so each commit could take a second or longer (estimate, to be measured). Users running `git status` in such a checkout will also find it slow; git's fsmonitor and untracked cache help.
>
> *Options:* (a) accept the cost; (b) build commits with git plumbing or libgit2, which writes only the changed objects and trees without touching the index file; (c) reduce the number of commits with batch writes (1.0110).
>
> *Recommendation:* Set a target (for example under 200 ms per write at the target size), benchmark early, and plan for (b), combined with (c).

TODO: answer pending

## Compatibility

> **3.0010**: Which OS-level file-watching mechanisms are available, and is the design portable across operating systems?

inotify on Linux, FSEvents on macOS, ReadDirectoryChangesW on Windows. A watchdog-style library abstracts all three over the same OS primitives, so the design is the same regardless of the chosen language. The binding constraint is Linux/inotify, which matters because the backend server is most likely deployed on Linux.

> **3.0020**: Do native file watches work reliably over network filesystems?

No — not on NFS or some SMB setups. There, the periodic poll becomes the primary detection mechanism rather than a backstop.

> **3.0030**: Which operating systems and filesystems must the server support in the first release?
>
> *Context:* The specs say the server is most likely deployed on Linux and treat network filesystems as possible. Supporting NFS or SMB means poll-only change detection there, with slower polls. Supporting the server on Windows or macOS adds test effort, although the watch library hides most of the differences.
>
> *Recommendation:* Linux with a local filesystem for the first release; other platforms best effort.

TODO: answer pending

> **3.0040**: How are large or binary node attachments stored in git, and are they written through the API or only by hand?
>
> *Context:* Node attachments (REQ 41498f3b-3061-43d1-b910-44cc5d859409) are opaque files committed to git. Large binaries make the repository grow quickly and every clone slow; git LFS is the usual fix but needs extra tooling on every client.
>
> *Options:* (a) plain git with a size limit per file; (b) git LFS for attachments; (c) attachments stored outside git.
>
> *Recommendation:* (a) with a configurable limit for the first release.

TODO: answer pending

## Interaction Capability

> **4.0010**: Will individual edges be hand-edited directly and frequently?

Mostly via the API, but hand-editing is allowed and must not be dangerous. It does not need to be JSONL. This, together with the in-place attribute-update cost, drove the one-file-per-edge choice.

> **4.0020**: Must nodes be able to hold additional data files?

Yes. Nodes are represented as an id directory containing a convention-based index file (for example `index.<ext>` or a `node.*` glob), without changing the node/edge relationship. Sibling files are opaque payload: served on request, not parsed or indexed. Edges will not need this unless it later makes sense to unify.

> **4.0030**: What is the exact naming convention for the node index file?
>
> *Context:* This file holds the node's attributes; every other file in the node folder is an attachment. A fixed name (for example `node.yaml`) is simple and fast to find. A glob (`node.*`) allows different formats per node but makes lookup and validation more complex, and two matching files would be ambiguous. If 2.0110 moves the main file into the shard folder as `<id>.<ext>`, this question goes away.
>
> *Recommendation:* A fixed name, or the layout from 2.0110.

TODO: answer pending

> **4.0040**: Which query operations should the REST API and the MCP server offer?
>
> *Context:* The specs promise multi-hop traversal and similarity search but do not describe the query interface. For MCP clients (AI agents), a small set of clearly described tools is easier to use correctly than a general query language.
>
> *Options:* (a) a fixed set of operations: get by id, list by type with attribute filters, direct neighbors, traversal from a start node with maximum hops and type filters, similarity top-K; (b) raw read-only SQL against the index, which is powerful but makes the internal table layout a public contract and is hard to secure; (c) a graph query language (Cypher-like).
>
> *Recommendation:* (a) for the first release; (b) at most in the core library, not over REST or MCP.

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

> **5.0060**: How many GraphMgr server processes may run against one repository, and how is that enforced?
>
> *Context:* The in-process write lock (REQ 6d237d5a-0bab-4bce-b331-51d4a2c008c2) only works if exactly one process writes to a repository. An MCP server started over stdio runs one process per client, not per repository: two agent sessions on the same repository start two servers, each with its own lock, watcher and full rebuild. A REST server in a separate process, or a script that imports the core library, is another writer.
>
> *Options:* (a) one server per repository that hosts REST and MCP over HTTP; at startup it takes an OS file lock (for example `flock` on `.graphmgr/lock`), so a second instance refuses to start; a stdio MCP entry point is only a thin proxy to that server; (b) allow several processes and take a cross-process file lock for every write; each process still rebuilds its own index and runs its own watcher.
>
> *Recommendation:* (a). Affects REQ 6d237d5a-0bab-4bce-b331-51d4a2c008c2 and REQ 36c8e9c5-d0dc-4693-a3ec-41edc2811393; would become a new decision.

TODO: answer pending

> **5.0070**: How should API commits coexist with the user's own git work in the same checkout: uncommitted hand edits, and a manual `git commit`, `pull` or `rebase` running at the same moment?
>
> *Context:* Git allows one operation on its index at a time (`.git/index.lock`), so a manual git command running while the API commits makes one of them fail. If the API committed with `git add -A`, it would also commit the user's unrelated uncommitted edits. Independent of the per-type write lock, the commit step needs one global lock, because all commits share one git index.
>
> *Points to decide:* (a) the API commits only the paths it wrote itself; (b) when `index.lock` is held, retry for a short time, then either fail the write and undo the file change, or keep the file and commit later; (c) whether API writes pause while a merge or rebase is in progress.
>
> *Recommendation:* (a); for (b), retry and then undo the file change and report the failure; for (c), pause writes and report "busy".

TODO: answer pending

> **5.0080**: After an API write returns, must the next read through the API already see the change?
>
> *Context:* Step 7 of UC 98135519-b97e-4df0-8386-45ee5a2fb66d lets the change-detection pipeline pick up the API's own write, so the change becomes visible only after the debounce window or the next poll. The same use case also promises that the result is "immediately queryable".
>
> *Options:* (a) the write path updates the SQLite index itself before it returns, and the watcher ignores the events caused by the API's own writes; (b) visibility only through the watcher, eventually.
>
> *Recommendation:* (a).

TODO: answer pending

> **5.0090**: While a change made outside the API is still being synchronized, should queries return the last consistent state or wait for the sync?
>
> *Context:* Listed as open in UC 7f0637c2-9989-4d06-b1de-7c8a402aa17b and UC 247fe425-e8d7-4eea-b8aa-1740da4ed943. Waiting gives fresher results but makes query time depend on the poll; serving the last state is simple but can lag behind the disk.
>
> *Recommendation:* Serve the last consistent state, and include the index timestamp or a "changes pending" flag in responses.

TODO: answer pending

## Security

> **6.0010**: Do the REST API and the MCP server need authentication and authorization, and on which network interfaces may they listen?
>
> *Context:* The original request says "access would always go through the API, so access and concurrency is controlled", but no security requirement exists yet. Without authentication, anyone who can reach the port can change the graph and its git history. The answer depends on 0.0060.
>
> *Options:* (a) listen on localhost only, without authentication (single-user tool); (b) a static API token per client; (c) user accounts with roles, for example read-only versus write, possibly per type.
>
> *Recommendation:* (a) as the default, and (b) required as soon as the server listens on any other interface.

TODO: answer pending

> **6.0020**: Which git author and committer identity should API commits use, and what should the commit message contain?
>
> *Context:* The git history is the audit trail (GOL 1585066f-9fc7-44bb-a3be-3c5e6cf91cde). If every API commit uses one service identity, the history shows what changed but not who asked for it.
>
> *Options:* (a) one service identity; (b) the caller as author and the service as committer, which needs authentication (6.0010) to know the caller; (c) one service identity plus the caller named in the commit message. In all cases, define a fixed message format, for example operation, entity type and id.
>
> *Recommendation:* (c) for the first release, (b) once authentication exists.

TODO: answer pending

## Maintainability

> **7.0010**: Should all access surfaces share one implementation of the business logic?

Yes. One shared core library (the ABI) implements validation, locking, file I/O, git commit, index sync, and queries. REST (for example FastAPI) and MCP are thin, logic-free transport wrappers around that single core — never duplicating business logic per transport, the same discipline specmgr itself follows.

> **7.0020**: What happens to existing entities when a type's JSON Schema changes?
>
> *Context:* Every entity carries `schema_version`, but no rule says what it means. Adding a required field to a schema would make every existing entity of that type invalid at the next start. It is also open whether a schema change is itself a write through the API, with its own commit.
>
> *Options:* (a) schemas may only change in compatible ways (new optional fields); a breaking change needs a new type; (b) a breaking change raises the schema version and comes with a migration that rewrites the affected files in one commit; (c) old schema versions are kept, and each entity is validated against the version it declares.
>
> *Recommendation:* (a) or (c) for the first release; define the meaning of `schema_version` in a requirement.

TODO: answer pending

> **7.0030**: Should the project use "edge" or "link" as its single term?
>
> *Context:* The docs and the layout mix both: the data folder is `links/`, the schema folder is `schemas/edge/`, the SQLite table is `links`, the problem statement says "Node/Link" and the goals say "Node/Edge". One term makes API names, MCP tool names and docs easier to follow.
>
> *Recommendation:* "edge" (the standard graph term), including renaming the data folder to `edges/`.

TODO: answer pending

## Flexibility

> **8.0010**: Should git be the source of truth for change detection?

No. Detection is pure filesystem content tracking; the same mechanism would work pointed at a non-git directory. Git remains the history, rollback, and audit layer.

> **8.0020**: Is sharded JSONL still an option for the future?

Yes — as a targeted opt-in optimization for a specific edge type that is measured to be high-volume, append-dominant, and rarely updated. It is not the baseline; it would be adopted per edge type, not for all edges.

> **8.0030**: Can similarity or related-node search be added without changing the storage layout?

Yes, and it is additive. Embed each entity's name plus description with a small local model (fastembed / bge-small class, as precedent exists in specmgr's own find_related and find_similar_text tools), store the embeddings as another derived, rebuildable index via the sqlite-vec extension inside the same SQLite database (one engine, one lifecycle; an external vector-DB service was rejected as an inconsistent operational footprint; an embedded ANN library such as faiss or hnswlib is the fallback if sqlite-vec proves insufficient). The embedding sync piggybacks on the same change-detection pipeline, and embeddings are recomputed on every startup, consistent with the no-persisted-derived-state rule.

## Safety

## More Information

Source documents: first-conversation.md (curated record, in docs/history/) and session-first-conversation-ses_ed8c.md (raw transcript, kept only locally in the repository root and not committed), from session ses_ed8c0da78ffeLB78W5hsXJNOor.

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

On 2026-11-22, after a review of the specification set, questions 0.0060-0.0070, 1.0080-1.0130, 2.0080-2.0120, 3.0030-3.0040, 4.0040, 5.0060-5.0090, 6.0010-6.0020 and 7.0020-7.0030 were added, and context was added to the pending questions 1.0060, 1.0070 and 4.0030. Once answered, the answers are to be carried into new or superseding decisions and the affected requirements, use cases and verification records.

The Safety category is intentionally empty: neither the original discussion nor the 2026-11-22 review found safety-relevant questions for this system. The original discussion elicited no security requirements; the Security questions were added in the 2026-11-22 review.
