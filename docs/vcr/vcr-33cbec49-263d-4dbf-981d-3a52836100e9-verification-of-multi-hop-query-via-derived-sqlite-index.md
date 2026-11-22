---
classification: null
created: '2026-10-13T04:19:50.895+02:00'
id: 33cbec49-263d-4dbf-981d-3a52836100e9
status: draft
type: vcr
updated: '2026-10-13T04:19:50.895+02:00'
version: 1.0.0
---

# Verification of Multi-Hop Query via Derived SQLite Index

## Verifies

REQ 4d76e656-e36b-4b3a-9ce6-bc016f5f2f0f: Multi-Hop Query via Derived SQLite Index

This record defines how the multi-hop traversal capability is verified against the SQLite relational index, including the guarantee that the index is a derived, rebuildable cache and that the git working tree remains the sole source of truth.

## Coverage

partial

## Acceptance Criteria

### AC-001 (Demonstration): Known chain is traversed to the correct hop count

A hand-built chain of four nodes connected by three edges is traversed from the first node, returning the last node with the correct hop count and path.

#### Test Steps

1. Create nodes A, B, C, D and edges A-B, B-C, C-D through the API.
2. Run a multi-hop traversal from A with a maximum of three hops.
3. Assert that D is returned with hop count 3 and that the returned path lists A, B, C, D in order.

### AC-002 (Test): Planted paths in a 1000-node graph are recovered exactly

On a randomly generated 1000-node graph with a known set of planted paths, the traversal query returns exactly the planted reachable set: no false positives, no false negatives.

#### Test Steps

1. Generate a 1000-node test corpus with planted multi-hop paths and import it (API or direct file creation with sync).
2. Run the traversal query from each planted source.
3. Compare the result sets programmatically against the planted paths.
4. Assert zero false positives and zero false negatives.

### AC-003 (Inspection): Index is derived and rebuildable; working tree is the sole source of truth

Deleting the SQLite index file and rebuilding it from the text tree reproduces identical query results, confirming the index is never authoritative.

#### Test Steps

1. Record the result of a fixed traversal query on the running index.
2. Delete the transient SQLite index while the process is stopped.
3. Restart the process and rerun the same query.
4. Assert that the result is identical to the recorded one.

## More Information

Drafted 2026-10-13 as part of the initial specification set; harder algorithms (weighted shortest path, centrality) are out of scope for this record and may be layered later (for example via networkx over a materialized subgraph).

## Updates

<!-- Newest entry first -- prepend new entries directly below this comment. -->

### 2026-10-13 04:15:00.000+02:00 - Created

Verification case drafted from the accepted query-layer decision; coverage is none pending implementation.
