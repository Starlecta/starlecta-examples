# Connect an assistant to the hosted Starlecta MCP server

Server: `https://starlecta.com/mcp` (Streamable HTTP, stateless). Seven tools (server version 1.1.0): `lookup_influencer`, `compare_influencers`, `search_influencers`, `ai_search_influencers`, `audit_influencer`, `trend_cloud`, `brand_of_the_day`. A key is optional: without one, calls are tied to your network. The files named below hold the keyless setup; the `*-with-key` files next to them send your own key as `Authorization: Bearer slk_...` and get limits of their own. Own key: `POST https://starlecta.com/api/agent/keys` or the form at https://starlecta.com/en/agents/key (details in [auth.md](https://starlecta.com/auth.md)). The keyed files use the placeholder `slk_YOUR_KEY`; never commit a real key.

(These configs are for the hosted server. The local, offline server in [../../mcp-server](../../mcp-server/) is configured in [its own guide](../../mcp-server/examples/client-configs.md).)

| Client | File here | Where it goes |
|---|---|---|
| Cursor | [cursor.mcp.json](cursor.mcp.json) | `~/.cursor/mcp.json` (all projects) or `.cursor/mcp.json` (one project) |
| VS Code | [vscode.mcp.json](vscode.mcp.json) | `.vscode/mcp.json` in the workspace (the keyed variant makes VS Code ask for the key once and store it) |
| Claude Desktop, macOS and Linux | [claude-desktop.json](claude-desktop.json) | `claude_desktop_config.json`; needs Node for `npx` |
| Claude Desktop, Windows | [claude-desktop-windows.json](claude-desktop-windows.json) | `%APPDATA%\Claude\claude_desktop_config.json` |
| Claude Code | command below, or [claude-code.mcp.json](claude-code.mcp.json) | `claude mcp add`, or `.mcp.json` in the project root |

Formats checked on 2026-10-30 against the clients' own documentation: [Cursor](https://cursor.com/docs/context/mcp), [VS Code](https://code.visualstudio.com/docs/agents/reference/mcp-configuration), [Claude Code](https://code.claude.com/docs/en/mcp), [mcp-remote](https://github.com/geelen/mcp-remote). Client formats change; if one stops loading, the client's documentation wins.

## Cursor

[cursor-with-key.mcp.json](cursor-with-key.mcp.json) holds the key as text. To keep the key out of the file, set `STARLECTA_AGENT_KEY` in your environment and use Cursor's interpolation (`${env:NAME}` works in `url` and `headers`):

```json
"headers": { "Authorization": "Bearer ${env:STARLECTA_AGENT_KEY}" }
```

## VS Code

[vscode.mcp.json](vscode.mcp.json) and [vscode-with-key.mcp.json](vscode-with-key.mcp.json) use VS Code's own `servers` key (not `mcpServers`), `"type": "http"`; the keyed one adds an `inputs` prompt with `password: true`, so no key sits in the file. Input variables are a VS Code feature; they do not work in other clients' files. To put the key in the keyed file instead, replace the header value with `"Bearer slk_YOUR_KEY"` and delete the `inputs` block.

## Claude Desktop

Claude Desktop starts local servers, so the hosted one goes through the [`mcp-remote`](https://github.com/geelen/mcp-remote) bridge: `npx mcp-remote https://starlecta.com/mcp --header "Authorization: Bearer ${STARLECTA_KEY}"`. [claude-desktop.json](claude-desktop.json) is that command as a config without the header; [claude-desktop-with-key.json](claude-desktop-with-key.json) adds it, with the key in `env`. Restart Claude Desktop after editing.

On Windows, a space inside an `args` entry is not escaped, which breaks the header. [claude-desktop-windows-with-key.json](claude-desktop-windows-with-key.json) removes the space from the argument (`Authorization:${STARLECTA_AUTH}`) and puts it in the `env` value (`Bearer slk_YOUR_KEY`), where spaces are safe. This follows the workaround in the `mcp-remote` README. [claude-desktop-windows.json](claude-desktop-windows.json) needs no workaround because it sends no header.

## Claude Code

```sh
claude mcp add --transport http starlecta https://starlecta.com/mcp
```

With your own key:

```sh
claude mcp add --transport http starlecta https://starlecta.com/mcp \
  --header "Authorization: Bearer $STARLECTA_AGENT_KEY"
```

This expands the key from your shell and stores it in your local Claude Code configuration (local scope by default; `--scope user` or `--scope project` change that). For a project file you can share, use [claude-code.mcp.json](claude-code.mcp.json) (keyless) or [claude-code-with-key.mcp.json](claude-code-with-key.mcp.json), which reads `${STARLECTA_AGENT_KEY}` from each person's environment. Check with `claude mcp list`.

## If it does not work

- The MCP server answers a wrong or revoked key with HTTP 200 and an error result (`isError: true`), not with HTTP 401. Only the REST endpoints answer 401. Clients show this as a tool error. If a keyed setup fails, try the keyless file to tell a key problem from a connection problem.
- Rate limits are per key, or per network without one: 60 requests a minute across all tools, and daily limits per tool. See [auth.md](https://starlecta.com/auth.md).
- `trend_cloud` and `brand_of_the_day` never need a key; use either to check the connection.
