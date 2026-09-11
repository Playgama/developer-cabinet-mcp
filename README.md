# Playgama Developer Cabinet MCP Server

Publish and manage your HTML5 games on [Playgama](https://playgama.com/mcp/) straight from your
AI agent — Codex, Claude Code, Cursor or VS Code. Full guide: [Playgama wiki](https://wiki.playgama.com/playgama/mcp).

The server is hosted by Playgama. There is nothing to install or run — you connect a client to
the endpoint below with your token.

| | |
| --- | --- |
| Endpoint | `https://developer.playgama.com/api/mcp` |
| Transport | Streamable HTTP |
| Authentication | `Authorization: Bearer YOUR_TOKEN` |
| Token | Issue it at [developer.playgama.com/mcp](https://developer.playgama.com/mcp) |

You need a Playgama developer account with the sign-up finished. A token reaches the games of
your own organization and nothing else. It does not expire — revoke it on the same page.

Questions: [developer.success@playgama.com](mailto:developer.success@playgama.com)

## Connect

Replace `YOUR_TOKEN` with your token.

### Codex

`~/.codex/config.toml`:

```toml
[mcp_servers.playgama-developer-cabinet]
url = "https://developer.playgama.com/api/mcp"
http_headers = { "Authorization" = "Bearer YOUR_TOKEN" }
```

### Claude Code

```sh
claude mcp add --transport 'http' 'playgama-developer-cabinet' 'https://developer.playgama.com/api/mcp' --header 'Authorization: Bearer YOUR_TOKEN'
```

### Cursor

`~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "playgama-developer-cabinet": {
      "type": "http",
      "url": "https://developer.playgama.com/api/mcp",
      "headers": {
        "Authorization": "Bearer YOUR_TOKEN"
      }
    }
  }
}
```

### VS Code

`.vscode/mcp.json` — VS Code asks for the token once and keeps it out of the file:

```json
{
  "servers": {
    "playgama-developer-cabinet": {
      "type": "http",
      "url": "https://developer.playgama.com/api/mcp",
      "headers": {
        "Authorization": "Bearer ${input:playgama-developer-cabinet-mcp-token}"
      }
    }
  },
  "inputs": [
    {
      "id": "playgama-developer-cabinet-mcp-token",
      "type": "promptString",
      "description": "Playgama MCP token for playgama-developer-cabinet",
      "password": true
    }
  ]
}
```

### Claude Desktop and claude.ai

Not supported yet: their connectors require OAuth, and the server authenticates with a static
token.

## Tools

The authoritative list is what the server answers to `tools/list`. As of version 1.0.0:

| Tool | What it does | Kind |
| --- | --- | --- |
| **Games** | | |
| `list_applications` | Lists your organization's games, newest first | read-only |
| `create_application` | Creates a new game as a draft | write |
| `get_application` | Reads a game's saved form, archives and media | read-only |
| `update_application_form` | Saves form fields; fields you leave out keep their values | write |
| `get_submission_state` | Tells whether the game can be submitted to moderation now, and why not | read-only |
| `list_moderation_comments` | Reads the moderation correspondence on a game | read-only |
| **Builds** | | |
| `start_archive_upload` | Starts a zip upload and answers a one-hour upload URL | write |
| `confirm_archive_upload` | Adds the uploaded archive to the form and starts unpacking | write |
| `get_archive_status` | Reads unpacking progress and the Bridge SDK analysis | read-only |
| **Covers** | | |
| `start_cover_upload` | Starts a cover upload for one slot: square, portrait or landscape | write |
| `confirm_cover_upload` | Checks the image and puts it in its slot | write |
| **In-app purchases** | | |
| `list_in_app_products` | Reads the in-app catalog with its checksum | read-only |
| `replace_in_app_products` | Replaces the whole catalog; products left out are deleted | write |
| **Leaderboards** | | |
| `list_leaderboards` | Lists a game's leaderboards | read-only |
| `create_leaderboard` | Adds a leaderboard | write |
| `update_leaderboard` | Changes a leaderboard's name, type or score order | write |
| **Testing** | | |
| `get_archive_qa_tool_link` | Opens an uploaded build in the Playgama QA Tool | read-only |
| `get_local_game_qa_tool_link` | Opens a game served from localhost in the QA Tool | read-only |
| **Sandbox** | | |
| `get_sandbox_state` | Reads what is live in the sandbox and whether a publish would be accepted | read-only |
| `publish_sandbox` | Makes a build playable by anyone with the link, without moderation | write |

Every write tool is annotated `destructiveHint: true`, so clients ask before calling it.

### Uploading a build

1. `start_archive_upload` answers `uploadUrl` and `headers`.
2. PUT the zip to `uploadUrl` with exactly those headers, e.g.
   `curl -T game.zip -H "Content-Type: application/zip" "<uploadUrl>"`.
3. `confirm_archive_upload`, then poll `get_archive_status` until `processing` is `DONE` or
   `FAILED`.

Covers work the same way, one slot per call: a PNG or JPEG of exactly 800×800 (square),
1080×1920 (portrait) or 1920×1080 (landscape).

## Limits

- Build archive: up to 300 MB per zip.
- Cover: up to 10 MB per file.
- Sandbox: 3 publications per rolling hour per game; only publications that change something count.

## What the server deliberately does not do

These stay human actions in the cabinet:

- submitting a game to moderation;
- deleting anything — leaderboards, or archives and covers from the form;
- rolling a game back to its last submitted version;
- reading payouts;
- uploading screenshots, videos and other assets.

## License

[MIT](LICENSE)
