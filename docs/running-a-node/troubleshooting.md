---
title: Troubleshooting
description: Common Kakitu node issues and how to resolve them
---

# Troubleshooting

This page covers the most common issues encountered when running a Kakitu node.

---

## Node Not Syncing

**Symptom**: `block_count` is not increasing, `unchecked_count` stays high, `peer_count` is 0 or very low.

**Checks**:

1. Verify port 7075 is open and reachable:
   ```bash
   curl -s -d '{"action":"peers"}' http://localhost:7076
   ```
   If the peer list is empty, the node cannot reach the network.

2. Check your firewall:
   ```bash
   ufw status
   # Port 7075/tcp should be ALLOW
   ```

3. Check if the node process is running:
   ```bash
   docker ps | grep kakitu
   # or
   systemctl status kakitu-node
   ```

4. Check logs for errors:
   ```bash
   docker logs --tail 200 kakitu-node
   ```

5. Try a manual bootstrap:
   ```bash
   curl -s -d '{"action":"bootstrap_any"}' http://localhost:7076
   ```

---

## RPC Not Responding

**Symptom**: `curl` commands to port 7076 time out or return `Connection refused`.

**Checks**:

1. Verify the node is running:
   ```bash
   docker ps | grep kakitu
   ```

2. Check RPC bind address in `config-rpc.toml`:
   ```toml
   [rpc]
   address = "::ffff:127.0.0.1"
   port = 7076
   ```

3. Test from inside the Docker container:
   ```bash
   docker exec kakitu-node curl -s -d '{"action":"version"}' http://127.0.0.1:7076
   ```

4. Check logs for `RPC` errors:
   ```bash
   docker logs kakitu-node 2>&1 | grep -i rpc
   ```

---

## "Block not found" Errors

**Symptom**: RPC returns `{"error": "Block not found"}` for a hash you expect to exist.

**Cause**: The node may not have received this block yet, or it may be unchecked (pending confirmation).

**Checks**:

1. Check if the block is in the unchecked table:
   ```bash
   curl -s -d '{
     "action": "unchecked_get",
     "hash": "BLOCK_HASH"
   }' http://localhost:7076
   ```

2. Wait for the node to fully sync and retry.

3. Use `block_info` with `json_block: true` for more detail:
   ```bash
   curl -s -d '{
     "action": "block_info",
     "hash": "BLOCK_HASH",
     "json_block": "true"
   }' http://localhost:7076
   ```

---

## High unchecked_count

**Symptom**: `unchecked` value in `block_count` is very large (>10,000).

**Cause**: The node is bootstrapping or recovering from a network partition. This is normal during initial sync.

**Resolution**: Allow the node to continue syncing. Monitor `cemented_count` — as long as it increases, the node is making progress.

---

## Low Peer Count

**Symptom**: `peer_count` is consistently low (< 5).

**Checks**:

1. Ensure port 7075 TCP is open inbound and outbound
2. Check that the preconfigured peers are reachable:
   ```bash
   curl -s -d '{"action":"keepalive","address":"peering.kakitu.org","port":"7075"}' http://localhost:7076
   ```
3. Check for NAT/UPnP issues in your router if running at home

---

## Work Generation Failures

**Symptom**: `work_generate` returns an error or takes very long.

**Checks**:

1. Verify the work server is running:
   ```bash
   curl -s -d '{"action":"work_generate","hash":"0000000000000000000000000000000000000000000000000000000000000000"}' http://localhost:7076
   ```

2. Check work peer connectivity:
   ```bash
   curl -s -d '{"action":"work_peers"}' http://localhost:7076
   ```

3. If using OpenCL, verify GPU detection:
   ```bash
   docker exec kakitu-node kakitu_node --diagnostics
   ```

4. Fall back to CPU work temporarily by setting `work_threads = 2`

---

## Disk Full

**Symptom**: Node crashes, logs show I/O errors or LMDB write failures.

**Resolution**:

1. Stop the node immediately
2. Free disk space (delete old logs, snapshots, or unused files)
3. Restart the node

Prevent this by setting up disk usage alerts at 80% capacity. See [Monitoring](monitoring.md).

---

## Database Corruption

**Symptom**: Node fails to start with errors like `Failed to open database`, `MDB_CORRUPTED`.

**Resolution**:

1. Stop the node
2. Move the corrupted `data.ldb` file:
   ```bash
   mv ~/kakitu-ledger/data.ldb ~/kakitu-ledger/data.ldb.corrupted
   ```
3. Restore from a backup or download a snapshot
4. Restart the node

---

## Version Mismatch

**Symptom**: Node connects but immediately disconnects peers, or fails to process blocks.

**Cause**: Running an outdated node version incompatible with the current network protocol.

**Resolution**: Upgrade to the latest release. See [Node Releases](../releases/node-releases.md).

---

## Getting Help

If your issue is not covered here:

- Check the [GitHub Issues](https://github.com/kakitucurrency/kakitu-node/issues)
- Ask in the [Kakitu Discord](https://discord.gg/kakitu)
- Post in the community forum at [kakitu.org/community](https://kakitu.org/community)

When reporting issues, include:
- Node version (`{"action":"version"}` output)
- OS and Docker version
- Relevant log lines
- Steps to reproduce
