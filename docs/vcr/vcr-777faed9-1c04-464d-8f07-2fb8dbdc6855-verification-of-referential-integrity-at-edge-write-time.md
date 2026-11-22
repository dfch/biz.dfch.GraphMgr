---
classification: null
created: '2026-10-13T04:17:55.078+02:00'
id: 777faed9-1c04-464d-8f07-2fb8dbdc6855
status: draft
type: vcr
updated: '2026-10-13T04:17:55.078+02:00'
version: 1.0.0
---

# Verification of Referential Integrity at Edge Write Time

## Verifies

REQ c7f59a15-31f8-4f10-adc4-becd73deb675: Referential Integrity at Edge Write Time

This record defines how the write-time referential-integrity guarantee is verified: an edge whose src or dst does not resolve to an existing node must be rejected, while a valid edge must be written and committed. Because edges are first-class entities (not inline tags), this check is possible at write time and must hold for both endpoint directions.

## Coverage

partial

## Acceptance Criteria

### AC-001 (Test): Edge creation with a nonexistent dst is rejected

The API rejects a proposed edge whose dst does not resolve to an existing node, leaves no file on disk, and creates no git commit.

#### Test Steps

1. Start the API process against an empty repository.
2. Create node A through the API.
3. Attempt to create an edge of a valid type with src = A and dst = a UUID that does not exist.
4. Assert that the API returns a structured rejection and continues to serve requests.
5. Assert that no file exists under the links directory and that git log shows no new commit.

### AC-002 (Test): Edge creation with a nonexistent src is rejected

The same rejection applies to the src endpoint, confirming both directions are resolved.

#### Test Steps

1. Create node B through the API.
2. Attempt to create an edge with src = a UUID that does not exist and dst = B.
3. Assert that the API returns a structured rejection.
4. Assert that no edge file exists and no git commit was created.

### AC-003 (Test): Valid edge creation succeeds with exactly one commit

A proposed edge whose src and dst both resolve is written to its sharded file path and committed exactly once.

#### Test Steps

1. Create nodes A and B through the API.
2. Create an edge with src = A and dst = B.
3. Assert that exactly one edge file exists under the links directory, in the shard derived from the edge's own id.
4. Assert that git log shows exactly one new commit for this write.

### AC-004 (Inspection): Write path resolves both endpoints before any file write

Source inspection confirms that src and dst resolution against the node store happens before the edge file is written, for every transport (ABI, REST, MCP).

## More Information

Drafted 2026-10-13 as part of the initial specification set derived from the 2026-10-10/11 design discussion; no criterion has been executed yet (the system is not implemented).

## Updates

<!-- Newest entry first -- prepend new entries directly below this comment. -->

### 2026-10-13 04:15:00.000+02:00 - Created

Verification case drafted from the agreed architecture; coverage is none pending implementation.
