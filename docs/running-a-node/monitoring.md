---
title: Monitoring
description: Monitoring your Kakitu node with telemetry and metrics
---

# Monitoring

Keeping visibility into your node's health is essential, especially for Principal Representatives and services that rely on the node for transaction processing.

---

## Built-in Telemetry

The fastest way to check node health:

```bash
curl -s -d '{"action":"telemetry"}' http://localhost:7076
```

```json
{
  "block_count": "12345678",
  "cemented_count": "12345670",
  "unchecked_count": "8",
  "account_count": "34521",
  "bandwidth_cap": "10485760",
  "peer_count": "42",
  "protocol_version": "18",
  "uptime": "864000",
  "genesis_block": "GENESIS_HASH",
  "major_version": "1",
  "minor_version": "0",
  "patch_version": "0"
}
```

Key metrics to watch:

| Metric | Healthy value |
|---|---|
| `cemented_count` close to `block_count` | Indicates full sync |
| `unchecked_count` | Should be near 0; spikes are normal during bootstrapping |
| `peer_count` | Should be > 5; more is better |
| `uptime` | Continuously increasing |

---

## Block Count & Sync Status

```bash
curl -s -d '{"action":"block_count"}' http://localhost:7076
```

```json
{
  "count": "12345678",
  "unchecked": "8",
  "cemented": "12345670"
}
```

The node is fully synced when `cemented` is within ~1% of `count`.

---

## Confirmation Quorum

Check the current network voting state:

```bash
curl -s -d '{"action":"confirmation_quorum","peer_details":"true"}' http://localhost:7076
```

```json
{
  "quorum_delta": "67000000000000000000000000000000000000",
  "online_weight_minimum": "60000000000000000000000000000000000000",
  "online_weight_quorum_percent": "67",
  "online_stake_total": "95000000000000000000000000000000000000",
  "peers_stake_total": "90000000000000000000000000000000000000"
}
```

---

## Peer Connections

```bash
curl -s -d '{"action":"peers","peer_details":"true"}' http://localhost:7076
```

---

## Node Version

```bash
curl -s -d '{"action":"version"}' http://localhost:7076
```

```json
{
  "rpc_version": "1",
  "store_version": "21",
  "protocol_version": "18",
  "node_vendor": "Kakitu v1.0.0",
  "store_vendor": "LMDB",
  "network": "live",
  "network_identifier": "GENESIS_HASH",
  "build_info": "build info here"
}
```

---

## Statistics

```bash
curl -s -d '{"action":"stats","type":"counters"}' http://localhost:7076
```

Returns internal counters for messages, blocks, votes, and errors.

---

## WebSocket Monitoring

The WebSocket interface (`port 7078`) provides real-time event streaming. Enable it in `config-node.toml`:

```toml
[websocket]
enabled = true
address = "::ffff:127.0.0.1"
port = 7078
```

Subscribe to confirmation events:

```json
{
  "action": "subscribe",
  "topic": "confirmation",
  "options": {
    "confirmation_type": "active_quorum"
  }
}
```

Subscribe to all topics:

```json
{"action": "subscribe", "topic": "confirmation"}
{"action": "subscribe", "topic": "vote"}
{"action": "subscribe", "topic": "stopped_election"}
{"action": "subscribe", "topic": "active_difficulty"}
{"action": "subscribe", "topic": "telemetry"}
{"action": "subscribe", "topic": "new_unconfirmed_block"}
```

---

## Prometheus / Grafana Integration

The kakitu-node exposes Prometheus-compatible metrics when the stats endpoint is enabled. Use [kakitu-looker-server](https://github.com/kakitucurrency/kakitulooker-server) for a pre-built Grafana dashboard tailored to Kakitu.

### Basic Prometheus Scrape Config

```yaml
# prometheus.yml
scrape_configs:
  - job_name: kakitu_node
    static_configs:
      - targets: ['localhost:7076']
    metrics_path: /stats
    params:
      action: [stats]
      type: [counters]
```

---

## Alerting Recommendations

Set up alerts for:

- `cemented_count` stalls (not increasing for > 60 seconds under load)
- `peer_count` drops below 3
- Disk usage exceeds 80%
- Node process not running (systemd / Docker container exit)
- RPC endpoint not responding (HTTP health check)

Use tools like Alertmanager, UptimeRobot, or a simple cron-based health check script.
