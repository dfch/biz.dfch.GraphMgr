---
classification: null
created: '2026-10-13T04:19:24.124+02:00'
id: 25767710-01b6-47ce-bc55-1f2d347751ab
status: draft
type: vcr
updated: '2026-10-13T04:19:24.124+02:00'
version: 1.0.0
---

# Verification of Full Rebuild of Derived State on Process Start

## Verifies

REQ 06f32855-c050-4fcd-8538-a3550ccff070: Full Rebuild of Derived State on Process Start

This record defines how the rebuild-on-startup invariant is verified: on every process start, the change-detection manifest, the SQLite relational index, and the vector embeddings are rebuilt from the text files from scratch, and no derived state persists across restarts.

## Coverage

partial

## Acceptance Criteria

### AC-001 (Test): Restarted index matches the on-disk tree exactly

After a restart, the relational index contains exactly the entities present in the text tree: identical row counts for nodes and links, and identical values for sampled attributes.

#### Test Steps

1. Populate the tree and run the process until fully synchronized.
2. Record node and link row counts and the full attribute set of five sampled entities.
3. Stop the process and start it again.
4. Assert that row counts match and that the sampled entities return identical attribute values.

### AC-002 (Test): Changes made while stopped are present after start without incremental sync

A file modified while the process is stopped is reflected in the freshly built index after startup, with no reliance on any persisted manifest.

#### Test Steps

1. Stop the process.
2. Modify one node's index file on disk.
3. Start the process.
4. Assert that the modified value is queryable once the startup rebuild completes.

### AC-003 (Inspection): No persisted derived state exists across restarts

Inspection confirms that the manifest, the SQLite index file, and the embeddings are not read from or written to any persistent location at shutdown or startup, and that deleting the transient index location before start produces a correct rebuild.

#### Test Steps

1. Inspect the startup and shutdown code paths for any read of, or write to, persisted derived state.
2. Delete the transient index location while the process is stopped.
3. Start the process and assert that all entities are queryable with correct values.

## More Information

Drafted 2026-10-13 as part of the initial specification set; the accepted trade-off is a cold start that grows with corpus size (see the cold-start risk).

## Updates

<!-- Newest entry first -- prepend new entries directly below this comment. -->

### 2026-10-13 04:15:00.000+02:00 - Created

Verification case drafted from the accepted derived-state decision; coverage is none pending implementation.
