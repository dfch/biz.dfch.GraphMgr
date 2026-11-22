---
classification: null
created: '2026-10-13T03:52:04.972+02:00'
id: 2c4a32c5-c9a7-4ad7-8740-53288b3ff15b
status: accepted
type: dec
updated: '2026-10-13T04:00:46.693+02:00'
version: 1.0.0
---

# Overall Approach: Bespoke System Following the specmgr Pattern

## Context and Problem Statement

biz.dfch.GraphMgr must manage a generic, typed node/link network: nodes carry a fixed core attribute set (id, type, schema_version, created, updated) plus type-specific attributes, and links are typed, directed (src, dst), UML-connector-like entities with their own attributes. The whole structure must be controlled through an ABI, a REST API, and an MCP server, and every change must be committable to git for precise, reviewable history (see PRB e1dd6083-cbd6-42da-a285-bf9e80c21d5a: Generic Typed Node/Link Graph with Git-Trackable Text Storage). A research pass found no mainstream library that packages exactly this combination; the closest prior art (PKM tools, Structurizr DSL, TerminusDB/Dolt, DOT/GraphML/GEXF, RDF/Turtle + SPARQL, networkx/Kùzu as a derived index) each misses at least one constraint.

## Decision Drivers

- Git-native text storage: every node and edge must be a plain-text file diffable and reviewable with standard git tooling.
- Generic, extensible model: not tied to one domain; new types must be addable without deploying new code.
- First-class typed edges: edges need their own id, type, direction, and type-specific attributes, and the API must enforce referential integrity at edge write time.
- Governed access: all writes go through one controlled path (ABI/REST/MCP) with locking and validation; direct manual editing must still be tolerated.
- Proven pattern: the specmgr system demonstrates file-per-entity, frontmatter core attributes, per-type validation, and thin transport layers in one fixed domain.

## Considered Options

Three paths were weighed: (1) build a bespoke system following the specmgr pattern, generalized to a generic typed graph with dedicated edge entities; (2) adopt a standards-based graph (RDF/Turtle + SPARQL, e.g. rdflib or Oxigraph); (3) adopt Structurizr DSL if the domain were purely software architecture. Within the chosen path, two sub-decisions were made: edges as first-class stored entities versus inline reference tags inside node files, and type schemas as data-driven JSON Schema files versus code-driven Pydantic models per type.

## Decision Outcome

Option 1 is chosen: a bespoke system following the generalized specmgr pattern. Edges are first-class, separately stored entities (not inline tags), which lets the API reject an edge whose src or dst does not resolve to an existing node at write time. Type schemas are data-driven JSON Schema files under schemas/node/ and schemas/edge/, validated generically: adding a type means adding a schema file, not shipping code.

### Consequences

- Maximum control over storage, validation, and concurrency, at the cost of building and maintaining the system itself.
- Hard referential integrity at edge write time, a deliberate departure from specmgr's soft, inline-tag integrity.
- Extensibility without code deployment, with slightly weaker static type-checking support than code-driven per-type models.
- The remaining engineering concerns (file layout, change detection, query index, concurrency) had to be settled explicitly; they are recorded in the sibling decisions of this group (storage layout, change detection, derived state and query layer).

### Confirmation

The user confirmed the bespoke path, the dedicated edge entities, and the data-driven JSON Schema files during the 2026-10-10/11 design discussion (first-conversation.md sections 1, 2, and 3).

## Roles and Responsibilities

### Accountable

The project owner (user of the 2026-10-10/11 design discussion).

### Responsible

- The GraphMgr development team (to be assigned at implementation start).

### Consulted

- The 2026-10-10/11 design discussion, including the assistant's prior-art research findings.

### Informed

- All future maintainers of the GraphMgr specification documents.

## Tags

- architecture
- approach
- extensibility

## Source

Session ses_ed8c0da78ffeLB78W5hsXJNOor, recorded in first-conversation.md sections 1-3 and in session-first-conversation-ses_ed8c.md (both at the repo root).

## Related Artifacts

### Goals

- GOL 86b457d9-58b0-4c9c-8c53-cf7849a72e75: Provide a Generic, Extensible Typed Node/Edge Graph Model
- GOL 1585066f-9fc7-44bb-a3be-3c5e6cf91cde: Keep Graph Data Precisely Trackable in Git

## Pros and Cons

### Option 1: Bespoke system following the specmgr pattern

Pros: exact match for all stated constraints (generic typed model, first-class typed edges, git-native text, ABI/REST/MCP); hard referential integrity achievable at write time; data-driven schemas make new types a configuration change. Cons: full build and maintenance cost; no inherited query engine or ecosystem; every engineering concern (layout, locking, detection, diff quality) must be designed explicitly.

### Option 2: RDF/Turtle + SPARQL (rdflib / Oxigraph)

Pros: standards-based typed graph model with a real query language; text-serializable. Cons: clean diffs only under disciplined stable serialization; steeper learning curve; does not deliver the file-per-entity git workflow or the dedicated first-class edge entities with their own attribute schemas; no packaged REST+MCP+ABI governance.

### Option 3: Structurizr DSL (C4 model)

Pros: typed elements plus typed, directed relationships in pure text in git, off the shelf, with CLI/API. Cons: domain-locked to software/system architecture; not a generic, extensible node/link model; no multi-surface (ABI/REST/MCP) governance of a mutable graph store.

## Updates

<!-- Newest entry first -- prepend new entries directly below this comment. -->

### 2026-10-13 03:52:00.000+02:00 - Created

Decision recorded from the 2026-10-10/11 design discussion; all three options were weighed and option 1 confirmed by the user.
