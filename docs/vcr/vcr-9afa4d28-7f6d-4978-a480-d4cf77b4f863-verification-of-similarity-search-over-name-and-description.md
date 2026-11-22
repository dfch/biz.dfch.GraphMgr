---
classification: null
created: '2026-10-13T04:20:26.769+02:00'
id: 9afa4d28-7f6d-4978-a480-d4cf77b4f863
status: draft
type: vcr
updated: '2026-10-13T04:20:26.769+02:00'
version: 1.0.0
---

# Verification of Similarity Search over Name and Description

## Verifies

REQ 64d52fab-3260-4d96-aefe-a2e2aa4ad11f: Similarity Search over Name and Description

This record defines how the vector-similarity capability is verified: top-K nearest-neighbor search over locally computed embeddings of entity names and descriptions, stored via sqlite-vec in the same SQLite database, with no external API calls.

## Coverage

partial

## Acceptance Criteria

### AC-001 (Demonstration): Query text returns the semantically closest entity as top-1

Given entities with clearly distinguishable names and descriptions, a query text close to one entity's description returns that entity as the top result with a higher score than the others.

#### Test Steps

1. Create at least five entities with distinct, topically separable names and descriptions through the API.
2. Submit a similarity query whose text is close to one entity's description, with K = 3.
3. Assert that the intended entity is the top-1 result.
4. Assert that its similarity score exceeds the scores of the other returned results.

### AC-002 (Test): Type filter combines with similarity results

A similarity query restricted to one entity type returns only entities of that type, ranked consistently with the unfiltered expectation within the type.

#### Test Steps

1. Create entities of two different types whose names and descriptions are all semantically similar to the query text.
2. Submit the similarity query with a type filter for one of the types, K = 5.
3. Assert that every returned entity has the filtered type.
4. Assert that the relative order matches the unfiltered ranking restricted to that type.

### AC-003 (Inspection): Embedding computation is fully local, with no external API calls

Running the embedding pipeline under a network trace shows no outbound connections to external services; embeddings are produced by a local model.

#### Test Steps

1. Start a network trace on the host running the API process.
2. Trigger embedding computation (process start or a changed entity file).
3. Stop the trace and assert that no outbound requests were made to external endpoints during the embedding window.

### AC-004 (Test): Embeddings track content changes through the same sync pipeline

Modifying an entity's name or description changes its ranking on subsequent similarity queries, confirming that embeddings are recomputed on detected content changes.

#### Test Steps

1. Record the top-3 similarity results for a fixed query.
2. Modify the description of one entity so that it becomes clearly the closest match to the query; run no git commands.
3. Wait for the next detection cycle.
4. Rerun the similarity query and assert that the modified entity is now top-1.

## More Information

Drafted 2026-10-13 as part of the initial specification set; the embedding model and exact vector dimensions are implementation choices constrained to be local and self-hosted.

## Updates

<!-- Newest entry first -- prepend new entries directly below this comment. -->

### 2026-10-13 04:15:00.000+02:00 - Created

Verification case drafted from the accepted query-layer decision; coverage is none pending implementation.
