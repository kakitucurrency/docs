---
title: Node Security
description: Security best practices for Kakitu node operators
---

# Node Security

Security is the foundation of a well-run Kakitu node. Before deploying a node on any publicly reachable machine, review and apply all of the recommendations in this guide.

---

## RPC Security

The Kakitu node exposes an HTTP RPC interface on port **7076** (main network). This port must be **strictly controlled**.

### enable_control

The `enable_control` flag in the RPC configuration activates privileged RPC commands including:

- Stopping the node
- Wallet operations (creating seeds, sending funds)
- Work generation requests

!!! danger "Never expose enable_control to the public internet"
    Only enable `enable_control` when RPC access is restricted to `localhost` or a trusted internal network. Any process that can reach port 7076 with `enable_control = true` has full access to your wallets.

Configuration (`config-rpc.toml`):

```toml
[rpc]
enable_control = true  # Only safe when listening on localhost
address = "::ffff:127.0.0.1"
port = 7076
```

### Restrict RPC to localhost

Bind the RPC port to the loopback interface only:

```toml
[rpc]
address = "::ffff:127.0.0.1"
```

If you need remote RPC access, use an authenticated reverse proxy (e.g., nginx with TLS and an API key header) rather than exposing port 7076 directly.

---

## Network Port

The Kakitu node requires **TCP port 7075** to be open to the internet for peer-to-peer communication.

| Port | Protocol | Direction | Purpose |
|---|---|---|---|
| 7075 | TCP | Inbound + Outbound | Peer-to-peer live network |
| 7076 | TCP | Localhost only | RPC interface |
| 7078 | TCP | Localhost only | WebSocket interface |

Block all ports not listed above using your server's firewall.

---

## Firewall Configuration

### UFW (Ubuntu/Debian)

```bash
# Allow SSH (adjust port if needed)
ufw allow 22/tcp

# Allow Kakitu peer-to-peer
ufw allow 7075/tcp

# Block everything else by default
ufw default deny incoming
ufw default allow outgoing
ufw enable
```

### iptables

```bash
# Allow established connections
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Allow SSH
iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Allow Kakitu P2P
iptables -A INPUT -p tcp --dport 7075 -j ACCEPT

# Drop all other inbound
iptables -A INPUT -j DROP
```

---

## SSH Hardening

For nodes running on remote servers:

```bash
# /etc/ssh/sshd_config

# Disable root login
PermitRootLogin no

# Disable password authentication (use keys only)
PasswordAuthentication no

# Disable empty passwords
PermitEmptyPasswords no

# Set idle timeout (seconds)
ClientAliveInterval 300
ClientAliveCountMax 2
```

Generate and deploy an SSH key pair:

```bash
# On your local machine
ssh-keygen -t ed25519 -C "kakitu-node-operator"
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@your-server-ip
```

Install Fail2ban to block brute-force attempts:

```bash
apt install fail2ban
systemctl enable fail2ban
```

---

## Docker Security

When running the node in Docker:

- Use the official `kakitucurrency/kakitu-node` image
- Do **not** publish port 7076 to the host (`-p 7076:7076`)
- RPC access from Docker containers is restricted to the host machine by default
- Consider running the container as a non-root user

```bash
docker run \
  --name kakitu-node \
  -p 7075:7075 \
  -v ~/kakitu-ledger:/root \
  kakitucurrency/kakitu-node:latest
```

Note: `7076` is intentionally omitted from `-p` flags. Use `docker exec` or host-local HTTP for RPC access.

---

## Key and Seed Security

- **Never store seeds in plaintext on the node server**
- Back up seeds to offline, encrypted storage
- Do not use the node's built-in wallet for large amounts; consider external key management (see [Key Management](../integration-guides/key-management.md))
- Use a hardware security module (HSM) or encrypted key vault for production deployments

---

## Regular Maintenance

- Keep the OS patched: `apt update && apt upgrade`
- Upgrade to new Kakitu node releases promptly (see [Releases](../releases/node-releases.md))
- Monitor disk usage — the ledger grows continuously
- Rotate SSH keys periodically
- Review firewall rules after any infrastructure change
