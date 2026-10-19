# Setup

## The MCP server

Squirrel Notes' MCP server runs locally over stdio.

- **Words only:** `npx -y @squirrelnotes.app/mcp`.
- **With meaning search** (recommended for memory): `npx -y @squirrelnotes.app/mcp-embeddings`.
  - It is the same server with a small embedding model (gte-small, English) that runs on your machine; no memory text is sent anywhere to be embedded.
  - It's about a 77 MB download, plus ONNX Runtime.
  - Set `ONNXRUNTIME_NODE_INSTALL=skip` to avoid downloading GPU add-ons.

Environment:

| Variable | Meaning |
|---|---|
| `SQUIRREL_API_BASE_URL` | `https://api.squirrelnotes.app` |
| `SQUIRREL_API_KEY` | Your `sqn_…` key, from Settings → API in the app. The API needs a Pro account. |
| `SQUIRREL_PASSPHRASE` | Your encryption passphrase. The server derives your key from it locally. |
| `SQUIRREL_TIMEZONE` | Optional IANA zone (e.g. `Australia/Sydney`) for memory end dates, useful when the server runs on a machine set to UTC. |

Keep the key and passphrase in your MCP client's configuration or a secrets manager. Never type them on a command line, where shell history keeps them.

## Checking it works

Call `get_context`. A briefing (even an empty one) means the server unlocked your data. An error naming `SQUIRREL_API_KEY` means the key was rejected or the account isn't Pro. A network error names its cause, such as an untrusted certificate or a refused connection.
