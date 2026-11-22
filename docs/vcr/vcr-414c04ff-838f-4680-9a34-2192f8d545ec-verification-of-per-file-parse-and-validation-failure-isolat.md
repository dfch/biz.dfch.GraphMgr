---
classification: null
created: '2026-10-13T04:18:55.982+02:00'
id: 414c04ff-838f-4680-9a34-2192f8d545ec
status: draft
type: vcr
updated: '2026-10-13T04:18:55.982+02:00'
version: 1.0.0
---

# Verification of Per-File Parse and Validation Failure Isolation

## Verifies

REQ d20e1a61-7dad-4978-812b-53f35287acf4: Per-File Parse and Validation Failure Isolation

This record defines how the failure-isolation guarantee is verified: a file that fails to parse or fails schema validation must be marked invalid for that single entity only, without blocking synchronization of the rest of the graph, and must be retried on its next detected change.

## Coverage

partial

## Acceptance Criteria

### AC-001 (Test): Malformed file does not block synchronization of other entities

Writing syntactically invalid content into one node's index file leaves the process running, leaves all other entities queryable with correct results, and reports the affected entity as invalid or stale.

#### Test Steps

1. Start the API process against a populated tree with at least ten valid entities.
2. Overwrite one node's index file with syntactically invalid content.
3. Assert that the process is still running.
4. Assert that queries against the other nine entities return correct results.
5. Assert that the API reports the affected entity as invalid or stale.

### AC-002 (Test): Schema-invalid file is isolated without a process crash

Overwriting one node's index file with syntactically valid but schema-invalid content (for example a wrong attribute type) produces the same isolation behavior: no crash, other entities unaffected, the affected entity flagged.

#### Test Steps

1. Overwrite the node's index file with content that parses but violates the type's JSON Schema.
2. Assert that the process is still running and the other entities remain queryable.
3. Assert that the affected entity is reported invalid.

### AC-003 (Test): Repaired file resynchronizes on its next detected change

Restoring valid content to the previously invalid file causes the entity to become valid and queryable again, without a process restart.

#### Test Steps

1. Write schema-valid content back into the same node's index file.
2. Wait one poll interval plus the debounce window.
3. Assert that the entity is valid and queryable with the restored values.

### AC-004 (Inspection): Sync worker treats parse and validation failures as non-raising per-file conditions

Source inspection confirms that the synchronization loop catches per-file parse and schema-validation failures, records them as per-path conditions, and continues processing the remaining queue.

## More Information

Drafted 2026-10-13 as part of the initial specification set; this behavior is the mitigation for the out-of-band manual-edit corruption risk.

## Updates

<!-- Newest entry first -- prepend new entries directly below this comment. -->

### 2026-10-13 04:15:00.000+02:00 - Created

Verification case drafted from the accepted change-detection decision; coverage is none pending implementation.
