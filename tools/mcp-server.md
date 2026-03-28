# Kakitu MCP Server

**Repository:** [github.com/kakitucurrency/kakitu_mcp_server](https://github.com/kakitucurrency/kakitu_mcp_server)

The Kakitu MCP (Model Context Protocol) server exposes Kakitu network functionality to AI assistants such as Claude, enabling them to check balances, query accounts, send KSHS, and interact with the Kakitu network on behalf of users.

---

## What is MCP?

The [Model Context Protocol](https://modelcontextprotocol.io) is an open standard that allows AI assistants to call external tools and services in a structured, safe way. By running the Kakitu MCP server, you give AI assistants the ability to:

- Look up account balances and transaction history
- Query the network for block counts and confirmation status
- Generate `kshs_` addresses
- Construct and broadcast transactions (with user approval)

---

## Installation

```bash
git clone https://github.com/kakitucurrency/kakitu_mcp_server
cd kakitu_mcp_server
pip install -r requirements.txt
```

Or install via pip if published:

```bash
pip install kakitu-mcp-server
```

---

## Configuration

Create a `.env` file in the project root:

```env
KAKITU_RPC_URL=https://api.kakitu.org
# Optional: default account seed for transaction tools
KAKITU_SEED=your_seed_here
```

---

## Running the Server

```bash
python -m kakitu_mcp_server
```

The server starts and listens on stdio (standard MCP transport) or a configurable port.

---

## Connecting to Claude Desktop

Add the following to your Claude Desktop configuration (`~/Library/Application Support/Claude/claude_desktop_config.json` on macOS):

```json
{
  "mcpServers": {
    "kakitu": {
      "command": "python",
      "args": ["-m", "kakitu_mcp_server"],
      "cwd": "/path/to/kakitu_mcp_server"
    }
  }
}
```

Restart Claude Desktop. The Kakitu tools will appear in Claude's tool list.

---

## Available Tools

| Tool | Description |
|---|---|
| `kakitu_account_info` | Get balance, frontier, and representative for a `kshs_` address |
| `kakitu_block_count` | Get current block count and cemented count |
| `kakitu_pending` | List pending (unreceived) blocks for an account |
| `kakitu_send` | Send KSHS to a destination address |
| `kakitu_receive` | Receive all pending blocks for an account |
| `kakitu_generate_address` | Generate a new `kshs_` address from a seed |
| `kakitu_block_info` | Look up details of a specific block by hash |

---

## Example: Ask Claude

Once the MCP server is connected, you can ask Claude natural-language questions like:

> "What's the balance of kshs_1mqj8myiphp7uzzoopegxogwdqosrd4n9ybckp5kh3f8o1yuo974dywt7k7h?"

> "Send 2 KSHS to kshs_3ab7... from my account."

Claude will call the appropriate MCP tool and return the result inline in the conversation.

---

## Security Notes

- The MCP server can only send transactions if a seed is configured. For read-only use, omit the seed.
- Always review transaction details before confirming sends.
- Run the server locally; do not expose it to the public internet without authentication.

---

## Related

- [RPC Reference](../integration/rpc-reference.md)
- [kakitucurrency-js](./kakitucurrency-js.md)
- [gokakitu](./gokakitu.md)
