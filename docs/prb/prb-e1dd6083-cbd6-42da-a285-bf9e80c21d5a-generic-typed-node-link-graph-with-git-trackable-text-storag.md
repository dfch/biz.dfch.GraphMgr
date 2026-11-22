---
classification: null
created: '2026-10-13T03:48:10.551+02:00'
id: e1dd6083-cbd6-42da-a285-bf9e80c21d5a
status: draft
type: prb
updated: '2026-10-13T03:48:10.551+02:00'
version: 1.0.0
---

# Generic Typed Node/Link Graph with Git-Trackable Text Storage

The absence of an off-the-shelf system combining a generic typed node/edge graph with git-native text storage and ABI/REST/MCP access is causing the biz.dfch.GraphMgr project to remain unstarted, for the project owner because no surveyed prior-art candidate (binary database, PKM tool, architecture DSL, text graph format, standards-based graph) satisfies all required capabilities at once.

## Current State

### Summary

The required system does not exist as a single packaged product. A design discussion on 2026-10-10/11 surveyed the closest prior art and found partial matches only:

- PKM tools (Obsidian, Foam, Dendron, Logseq, Quartz): node = file, type = frontmatter field, link = wikilink index. Git-friendly and human-editable, but editor ecosystems without an ABI/REST/MCP-governed write path.
- Architecture-as-code DSLs (Structurizr DSL / C4): typed elements plus typed, directed relationships in pure text in git. Domain-locked to software/system architecture.
- "Git for data" databases (TerminusDB, Dolt): git semantics with a real query engine, but the data is not stored as freely editable text files on disk.
- Text-based graph interchange formats (DOT/Graphviz, GraphML, GEXF): genuinely plain text and diffable, but no typed schema system, no API layer, no concurrency control.
- Standards-based graphs (RDF/Turtle + SPARQL via rdflib/Oxigraph): a real typed graph model, text-serializable, but clean diffs only under disciplined stable serialization and a steeper learning curve.
- In-memory/embeddable graph engines (networkx, Kùzu): useful only as a derived index, not as storage.
- The `specmgr` system in the same environment: a working, domain-specific instance of the desired pattern (one markdown file per entity, YAML frontmatter core attributes, per-type validated body, reference tags resolved into directed links, thin MCP transport, per-domain write locks). Proof of feasibility for one fixed domain, not a general-purpose graph manager.

Binary databases are excluded by the core need: precise, reviewable change tracking of every node and edge through git, which line-based diffs on binary blobs cannot provide.

### What Is the Problem?

There is no single tool that manages a generic, extensible typed node/edge graph stored as plain-text, git-native files, with first-class typed directed edges, enforced referential integrity, multi-hop queries, and ABI/REST/MCP access surfaces.

### Why Is It a Problem?

Without it, graph data either lives in a binary database (no semantic diff, no hand-editability) or in ad-hoc text formats (no schema validation, no concurrency control, no query engine, no governed access).

### Where Is the Problem Observed?

In the `biz.dfch.GraphMgr` project, whose purpose is to provide exactly this system.

### Who Is Impacted?

The project owner and every future consumer of the node/link graph data.

### When Was the Problem First Observed?

2026-10-10, first design session (session `ses_ed8c0da78ffeLB78W5hsXJNOor`).

### How Is the Problem Observed?

A research pass found that every prior-art candidate misses at least one of the five required capabilities: generic typed model, first-class typed edges, text-native git storage, multi-surface API, multi-hop query.

## Gap

Zero packaged products satisfy all five capabilities simultaneously; the closest working example (specmgr) is domain-locked and uses inline reference tags instead of first-class edge entities, so it cannot enforce referential integrity at edge write time.

## Impact

The project cannot start without a settled architecture. The gap forces either adopting a partial tool (losing git-native text storage or generic typing) or building a bespoke system (higher cost, full control).

## Future State

A bespoke `biz.dfch.GraphMgr` system: a generic, extensible typed node/edge graph, one text file per entity in a git repository, all access through a shared core library (ABI) with thin REST and MCP transports, hard referential integrity at edge write time, a rebuildable SQLite query index with multi-hop traversal, vector similarity search over name and description, and git-independent filesystem change detection that absorbs direct manual edits.

## References

- `first-conversation.md` (repo root): curated record of the 2026-10-10/11 design discussion.
- `session-first-conversation-ses_ed8c.md` (repo root): raw session transcript.
