---
classification: null
created: '2026-10-13T04:03:49.232+02:00'
id: 6342b261-1cd0-4cdb-8ceb-70c284b8e2dd
status: open
type: rsk
updated: '2026-10-13T04:03:49.232+02:00'
version: 1.0.0
---

# Graph Corruption from Out-of-Band Manual Edits

## Cause

Direct manual editing outside the API is a supported write path. Editors may save a half-written file (save-as-rename produces CREATE + DELETE + MODIFY for one logical save), hand edits may violate the type's JSON Schema, and an out-of-band node deletion can leave edges whose src or dst no longer resolves (write-time referential integrity covers API writes only).

## Trigger

A malformed, mid-write, or schema-invalid file lands in the tracked tree, or a node with inbound or outbound edges is deleted manually.

## Consequence

Without isolation, a single bad file could block indexing of the whole graph, or queries could return stale or inconsistent results for entities that did not change.

## Scope

- Change-detection subsystem
- Query layer
- Graph data integrity

## Initial Assessment

### Probability 2

### Impact 3

## Strategy

reduce

## Mitigation

A 100-300 ms debounce window absorbs mid-save event bursts. Per-file, non-raising parse and validation isolation marks only the affected entity as invalid and retries it on its next detected change. The schema registry rejects invalid attribute values on API writes. The policy for edges dangling after an out-of-band node deletion (flag versus cascade) remains an open issue to resolve before implementation.

## Residual Assessment

### Probability 1

### Impact 2

## Owner

Development lead (to be assigned).

## Tags

- reliability
- data-quality

## Source

First design discussion, 2026-10-10/11 (first-conversation.md sections 3.1 and 4.3).

## More Information

See DEC 3cccd4cb-f488-4a6f-8005-f38d2b5e92ef: Change Detection: Git-Independent Filesystem Manifest with Hybrid Watch and Poll, and DEC 2c4a32c5-c9a7-4ad7-8740-53288b3ff15b: Overall Approach: Bespoke System Following the specmgr Pattern.
