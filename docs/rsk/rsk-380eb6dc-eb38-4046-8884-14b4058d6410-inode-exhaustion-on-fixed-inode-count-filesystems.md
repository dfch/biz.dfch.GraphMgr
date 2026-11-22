---
classification: null
created: '2026-10-13T04:03:01.303+02:00'
id: 380eb6dc-eb38-4046-8884-14b4058d6410
status: open
type: rsk
updated: '2026-10-13T04:03:01.303+02:00'
version: 1.0.0
---

# Inode Exhaustion on Fixed-Inode-Count Filesystems

## Cause

The one-file-per-entity plus folder-per-node storage model produces very large file and directory counts (100k nodes + 1M edges is roughly 1.1 million files plus one directory inode per node). Some filesystems, for example ext4 formatted with a conservative inode ratio, have a fixed total inode count set at format time.

## Trigger

The working tree grows until the filesystem's inode table is exhausted while free blocks remain (disk usage shows free space, inode usage is 100%).

## Consequence

New entities cannot be created. API writes and manual file creation fail until the filesystem is re-provisioned or the data is migrated to another filesystem.

## Scope

- Storage layer
- Deployment environment (filesystem choice and provisioning)

## Initial Assessment

### Probability 3

### Impact 3

## Strategy

reduce

## Mitigation

Document the deployment requirement: provision the backing filesystem with a generous inode budget (mkfs.ext4 -N) or choose a filesystem with dynamic inode allocation (XFS, APFS, NTFS). Monitor inode usage (df -i) in operational checks. Keep entity files small to limit growth per entity.

## Residual Assessment

### Probability 2

### Impact 2

## Owner

Deployment/operations lead (to be assigned).

## Tags

- storage
- deployment
- scale

## Source

First design discussion, 2026-10-10/11 (first-conversation.md section 5.6).

## More Information

See DEC a6235507-22fa-4aed-a45e-d9707c0f13bd: Storage Layout: One File per Entity, 256 Id-Prefix Shards. Raw disk-space inflation from block-size rounding (roughly 4.3 GB allocated for 100k nodes + 1M edges at 4 KB blocks) was assessed as a non-issue; inode exhaustion is the more likely failure mode.
