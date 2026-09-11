# Playgama Developer Cabinet MCP Server

A remote [Model Context Protocol](https://modelcontextprotocol.io) server for the
[Playgama Developer Cabinet](https://developer.playgama.com). It lets an AI agent — Claude Code,
Cursor, VS Code — work on your HTML5 games the way you do in the cabinet: create a game, fill in
its form, upload builds and covers, manage the in-app catalog and leaderboards, test in the
QA Tool and publish to a sandbox.

The server is hosted by Playgama. There is nothing to install or run — you connect a client to
the endpoint below with your token.

| | |
| --- | --- |
| Endpoint | `https://developer.playgama.com/api/mcp` |
| Transport | Streamable HTTP (stateless) |
| Authentication | `Authorization: Bearer pgm_mcp_…` |

## Access

Access is invite-only for now. To get a token, email
[developer.success@playgama.com](mailto:developer.success@playgama.com) with the email of your
Developer Cabinet account.

- A token reaches the games of your own organization and nothing else.
- A token does not expire. Revoke it from the same cabinet page you issued it on.

## Connect

Replace `pgm_mcp_YOUR_TOKEN` with your token.

### Claude Code

```sh
claude mcp add --transport http playgama-developer-cabinet https://developer.playgama.com/api/mcp \
  --header "Authorization: Bearer pgm_mcp_YOUR_TOKEN"
```

### Cursor

`~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "playgama-developer-cabinet": {
      "url": "https://developer.playgama.com/api/mcp",
      "headers": {
        "Authorization": "Bearer pgm_mcp_YOUR_TOKEN"
      }
    }
  }
}
```

### VS Code

`.vscode/mcp.json` — VS Code asks for the token once and keeps it out of the file:

```json
{
  "inputs": [
    {
      "id": "playgama-token",
      "type": "promptString",
      "description": "Playgama Developer Cabinet MCP token",
      "password": true
    }
  ],
  "servers": {
    "playgama-developer-cabinet": {
      "type": "http",
      "url": "https://developer.playgama.com/api/mcp",
      "headers": {
        "Authorization": "Bearer ${input:playgama-token}"
      }
    }
  }
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
| `list_applications` | Lists your organization's games, newest first | read |
| `create_application` | Creates a new game as a draft | write |
| `get_application` | Reads a game's saved form, archives and media | read |
| `update_application_form` | Saves form fields; fields you leave out keep their values | write |
| `get_submission_state` | Tells whether the game can be submitted to moderation now, and why not | read |
| `list_moderation_comments` | Reads the moderation correspondence on a game | read |
| **Builds** | | |
| `start_archive_upload` | Starts a zip upload and answers a one-hour upload URL | write |
| `confirm_archive_upload` | Adds the uploaded archive to the form and starts unpacking | write |
| `get_archive_status` | Reads unpacking progress and the Bridge SDK analysis | read |
| **Covers** | | |
| `start_cover_upload` | Starts a cover upload for one slot: square, portrait or landscape | write |
| `confirm_cover_upload` | Checks the image and puts it in its slot | write |
| **In-app purchases** | | |
| `list_in_app_products` | Reads the in-app catalog with its checksum | read |
| `replace_in_app_products` | Replaces the whole catalog; products left out are deleted | write |
| **Leaderboards** | | |
| `list_leaderboards` | Lists a game's leaderboards | read |
| `create_leaderboard` | Adds a leaderboard | write |
| `update_leaderboard` | Changes a leaderboard's name, type or score order | write |
| **Testing** | | |
| `get_archive_qa_tool_link` | Opens an uploaded build in the Playgama QA Tool | read |
| `get_local_game_qa_tool_link` | Opens a game served from localhost in the QA Tool | read |
| **Sandbox** | | |
| `get_sandbox_state` | Reads what is live in the sandbox and whether a publish would be accepted | read |
| `publish_sandbox` | Makes a build playable by anyone with the link, without moderation | write |

### Uploading a build

1. `start_archive_upload` answers `uploadUrl` and `headers`.
2. PUT the zip to `uploadUrl` with exactly those headers, e.g.
   `curl -T game.zip -H "Content-Type: application/zip" "<uploadUrl>"`.
3. `confirm_archive_upload`, then poll `get_archive_status` until `processing` is `DONE` or
   `FAILED`.

Archives are up to 300 MiB. Covers work the same way, one slot per call: a PNG or JPEG of exactly
800×800 (square), 1080×1920 (portrait) or 1920×1080 (landscape).

## Not available through MCP

These stay human actions in the cabinet:

- submitting a game to moderation;
- uploading screenshots, videos and other assets;
- removing an archive or a cover from the form;
- deleting a leaderboard;
- rolling a game back to its last submitted version.

## License

[MIT](LICENSE)
