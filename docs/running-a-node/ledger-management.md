---
title: Ledger Management
description: Managing the Kakitu node ledger database
---

# Ledger Management

The Kakitu ledger is a local copy of the entire Block Lattice — every account chain, every block, every vote record. Managing it correctly ensures node reliability and efficient operation.

---

## Ledger File

The ledger is stored as a single LMDB database file:

```
data.ldb
```

Located in your data directory (the volume mounted into the Docker container, or the platform-default data path for native installs).

---

## Ledger Size

The ledger grows continuously as new blocks are added. Current approximate sizes:

| Network | Size (approx.) |
|---|---|
| Main | ~8 GB (varies with adoption) |
| Test | ~100 MB |

Plan for continued growth. Use NVMe or SSD storage with at least 2× headroom.

---

## Bootstrapping (Initial Sync)

A new node must download the full ledger from peers. This process is called **bootstrapping**.

Monitor progress:

```bash
curl -s -d '{"action":"bootstrap_status"}' http://localhost:7076
```

```bash
curl -s -d '{"action":"block_count"}' http://localhost:7076
```

Compare your `cemented` count against network telemetry:

```bash
curl -s -d '{"action":"telemetry"}' http://localhost:7076
```

Bootstrapping on a fast connection typically completes within a few hours.

---

## Using a Ledger Snapshot

To avoid bootstrapping from scratch, you can download a ledger snapshot:

1. Download the latest snapshot from [snapshots.kakitu.org](https://snapshots.kakitu.org) (if available)
2. Stop the node
3. Replace `data.ldb` in your data directory with the snapshot file
4. Start the node — it will resume from the snapshot block count

!!! warning "Verify snapshot integrity"
    Always verify the SHA256 checksum of a downloaded snapshot before using it. Only use snapshots from trusted sources.

---

## Backup

```bash
# Stop the node first for a clean copy
docker stop kakitu-node

# Copy the ledger file
cp ~/kakitu-ledger/data.ldb ~/kakitu-ledger-backup/data.ldb.$(date +%Y%m%d)

# Restart the node
docker start kakitu-node
```

For live backups without stopping the node, use LMDB's `mdb_copy` tool:

```bash
docker exec kakitu-node mdb_copy /root /backup-path
```

---

## Pruning Unchecked Blocks

The node stores unchecked (unconfirmed) blocks in a temporary table. These are cleaned automatically but can accumulate during periods of network instability.

Check unchecked count:

```bash
curl -s -d '{"action":"block_count"}' http://localhost:7076
```

If `unchecked` is very large (>100,000), the node may be catching up from a network partition. Monitor and allow it to resolve naturally.

---

## Database Corruption Recovery

If the ledger becomes corrupted (node fails to start, reports database errors):

1. Stop the node
2. Delete `data.ldb` and `data.ldb-lock`
3. Restart — the node will re-bootstrap from scratch

Or restore from a backup or snapshot.

---

## Vacuum / Compaction

LMDB does not require manual compaction, but after deleting large amounts of data the free space within the file may not be returned to the OS. To reclaim space:

```bash
# Stop node first
docker stop kakitu-node

# Copy to a new compact file (mdb_copy with --compact)
mdb_copy --compact ~/kakitu-ledger ~/kakitu-ledger-compact

# Replace
mv ~/kakitu-ledger/data.ldb ~/kakitu-ledger/data.ldb.old
mv ~/kakitu-ledger-compact/data.ldb ~/kakitu-ledger/data.ldb

docker start kakitu-node
```
