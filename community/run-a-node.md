# Run a Kakitu Node

Running a Kakitu representative node helps secure and decentralise the network. Representatives participate in consensus (Open Representative Voting) by voting on blocks. The more weight delegated to independent representatives, the more resilient the network.

---

## What is a Representative?

In Kakitu's ORV consensus model, every account can delegate its voting weight to a **representative**. Representatives vote to confirm transactions. You don't need to hold any KSHS to run a representative — you just need a node that is online and reachable.

---

## Option A: Deploy on Railway (Easiest)

[Railway](https://railway.app) is a cloud platform that makes it easy to deploy Docker containers with persistent storage.

### Steps

1. Create a free Railway account at [railway.app](https://railway.app).
2. Click **New Project** → **Deploy from Docker Image**.
3. Enter the image: `kakitucurrency/kakitu-node:latest`.
4. Set environment variables:

   | Variable | Value |
   |---|---|
   | `KAKITU_NETWORK` | `live` |

5. Add a **Volume** mounted at `/root/KakituData` (minimum 20 GB).
6. Expose port `44075` (TCP + UDP) for P2P and `44076` (TCP) for RPC.
7. Deploy.

The node will start syncing. Check the logs in Railway's dashboard.

### Enable RPC on Railway

To use the RPC from outside Railway, expose port 44076 and add the following to your node config (via a mounted config file or environment variable):

```toml
[rpc]
enable = true
address = "::ffff:0.0.0.0"
port = 44076
enable_control = false   # keep false on public nodes
```

---

## Option B: Deploy on a VPS (Ubuntu 22.04)

### 1. Provision a Server

Recommended specs:

| Resource | Minimum |
|---|---|
| CPU | 2 vCPUs |
| RAM | 4 GB |
| Disk | 40 GB SSD |
| OS | Ubuntu 22.04 LTS |

Providers: DigitalOcean, Hetzner, Linode, Vultr, AWS EC2.

### 2. Open Firewall Ports

```bash
sudo ufw allow 44075/tcp
sudo ufw allow 44075/udp
sudo ufw allow 44076/tcp   # RPC (restrict to trusted IPs in production)
sudo ufw allow 44078/tcp   # WebSocket (optional)
sudo ufw enable
```

### 3. Install Docker

```bash
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER
```

Log out and back in, then verify:

```bash
docker --version
```

### 4. Run the Node

```bash
docker run -d \
  --name kakitu-node \
  --restart unless-stopped \
  -p 44075:44075/udp \
  -p 44075:44075/tcp \
  -p 44076:44076 \
  -v kakitu-data:/root/KakituData \
  kakitucurrency/kakitu-node:latest \
  kakitu_node daemon --network=live
```

### 5. Configure Peering

```bash
docker exec kakitu-node bash -c "
cat >> /root/KakituData/config-node.toml <<EOF
[node]
preconfigured_peers = [\"peering.kakitu.org:56111\"]
EOF
"
docker restart kakitu-node
```

### 6. Verify Sync

```bash
curl -d '{"action":"block_count"}' http://localhost:44076
```

The `cemented` count should increase over time.

---

## Promote Your Node as a Representative

Once your node is online and synced, share your node's `kshs_` address so other users can delegate their voting weight to you.

To find your representative address:

```bash
curl -d '{"action":"wallet_representative","wallet":"<your-wallet-id>"}' \
     http://localhost:44076
```

Announce it in the [Kakitu community](https://kakitu.org/community) so others can delegate to you.

---

## Monitoring

Check node status:

```bash
# Block count
curl -d '{"action":"block_count"}' http://localhost:44076

# Peer count
curl -d '{"action":"peers"}' http://localhost:44076

# Node version
curl -d '{"action":"version"}' http://localhost:44076
```

For production nodes, consider setting up [Uptime Kuma](https://github.com/louislam/uptime-kuma) or a similar monitoring tool to alert you if the node goes offline.

---

## Related

- [Install a Node](../getting-started/install-node.md)
- [Network Parameters](../network/network-parameters.md)
- [RPC Reference](../integration/rpc-reference.md)
