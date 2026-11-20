# lafo.trade MCP server

Remote MCP server for [lafo.trade](https://lafo.trade/en/), a service that rents out crypto trading robots for 3 or 6 months at a fixed price. The robots trade the renter's own exchange account through an API key with no withdrawal right.

This server lets a member's AI assistant **read** their robots, positions and history, and **pull a brake** when the member asks. It never places, changes or cancels an order, never closes a position and never restarts anything.

- Endpoint: `https://mcp.lafo.trade/mcp` (Streamable HTTP)
- Authentication: `Authorization: Bearer <key>` header. The key is created by the member on the "My AI" page of their lafo.trade account. Never put it in the URL.
- Official MCP Registry: `trade.lafo/robots`
- Documentation: https://lafo.trade/en/guide/assistant-ia

This repository holds documentation and client configuration only. The server is hosted by lafo.trade; there is nothing to install or run locally.

## Tools

### Reads

| Tool | What it returns |
|---|---|
| `get_my_account` | Your access, your exchange accounts and whether their robots are running |
| `get_my_positions` | Your open positions and the latest reading of each account, on all accounts or on one |
| `get_my_robots` | Your robots and their state |
| `get_robot_journal` | A robot's journal: what it did and why |
| `get_my_history` | Your closed positions and their result |

Every piece of data carries its age (`age_s`, `stale`). An assistant must quote it and never reason on stale data as if it were fresh.

### Brakes

| Tool | Effect |
|---|---|
| `brake_robot` | Pauses one robot |
| `brake_account` | Stops every robot on one exchange account |
| `brake_everything` | Stops all your robots |

A brake pauses. It does not cancel orders already placed at the exchange and closes no position: its answer says how many positions and orders remain in place. The assistant pulls a brake only when the member asks. Restarting is done by the member in the lafo.trade app.

### What it cannot do

- Place, change or cancel an order.
- Close a position, withdraw funds or restart a robot.
- Read someone else's account: a key only sees the account that created it.

## Keys and limits

- Two levels: "read", or "read + brakes".
- At most 5 active keys per member. "Revoke" cuts a key off immediately.
- At most 60 reads per minute and 10 brakes per hour, per key.
- Every call is logged on the member's "My AI" page.

## Connect a client

### Claude Code

```sh
claude mcp add --transport http lafo https://mcp.lafo.trade/mcp --header "Authorization: Bearer <key>"
```

### Cursor

`~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "lafo": {
      "url": "https://mcp.lafo.trade/mcp",
      "headers": { "Authorization": "Bearer <key>" }
    }
  }
}
```

### VS Code

`.vscode/mcp.json`:

```json
{
  "inputs": [
    { "type": "promptString", "id": "lafo-key", "description": "lafo.trade MCP key", "password": true }
  ],
  "servers": {
    "lafo": {
      "type": "http",
      "url": "https://mcp.lafo.trade/mcp",
      "headers": { "Authorization": "Bearer ${input:lafo-key}" }
    }
  }
}
```

## Who can use it

Members of lafo.trade with an active access. Without a valid key the server answers `401`.

Nothing returned by this server is investment advice.

## Links

- Website: https://lafo.trade/en/
- MCP guide: https://lafo.trade/en/guide/assistant-ia
- X: https://x.com/Lafotrade
