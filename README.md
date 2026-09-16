# Playgama Developer Cabinet MCP Server

Publish and manage your HTML5 games on [Playgama](https://playgama.com/mcp/) straight from your
AI agent — Codex, Claude Code, Cursor or VS Code. Full guide: [Playgama wiki](https://wiki.playgama.com/playgama/mcp).

The server is hosted by Playgama. There is nothing to install or run and no token to copy — you
give your client the endpoint below, sign in to the cabinet in the browser it opens, and allow
the connection.

## What you get

- **Publish in seconds** — ask your agent, get a public playable link back.
- **Get the audience** — the game takes its first players from the Playgama network, and
  Playgama DSP can send more the day it goes live (beta). The agent can start that campaign
  itself with `start_sandbox_traffic`; share the game first and the boost is free.
- **Start monetization** — rewarded, interstitial and banner ads switch on through
  [Playgama Ad](https://playgama.com/adv) once the game clears the session threshold
  (access by request).
- **Get all analytics** — playtime, retention and revenue reports in your developer
  dashboard; payouts start at 100 USD.

| | |
| --- | --- |
| Endpoint | `https://developer.playgama.com/api/mcp` |
| Transport | Streamable HTTP |
| Authentication | OAuth 2.1 — the client discovers it from the endpoint |
| Connected agents | See and revoke them at [developer.playgama.com/mcp](https://developer.playgama.com/mcp) |

You need a Playgama developer account with the sign-up finished. A connected agent reaches the
games of your own organization and nothing else. It stays connected while it keeps using the
server; revoke it on the same page and it is refused on its next call.

Questions: [developer.success@playgama.com](mailto:developer.success@playgama.com)

## Connect

Every client below opens the browser the first time it connects. Sign in to the cabinet if you
are not signed in, check the account and the agent, and allow it.

### Codex

```sh
codex mcp add 'playgama-developer-cabinet' --url 'https://developer.playgama.com/api/mcp'
```

### Claude Code

```sh
claude mcp add --transport 'http' 'playgama-developer-cabinet' 'https://developer.playgama.com/api/mcp'
```

Then run `/mcp` in Claude Code and choose the server to sign in.

### Cursor

`~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "playgama-developer-cabinet": {
      "type": "http",
      "url": "https://developer.playgama.com/api/mcp"
    }
  }
}
```

### VS Code

`.vscode/mcp.json`:

```json
{
  "servers": {
    "playgama-developer-cabinet": {
      "type": "http",
      "url": "https://developer.playgama.com/api/mcp"
    }
  }
}
```

### Claude Desktop and claude.ai

Settings → Connectors → Add custom connector, with the endpoint
`https://developer.playgama.com/api/mcp`.

### A token in a config from before OAuth

If your config still sends an `Authorization` header with a `pgm_mcp_` token, it keeps working
for now. Remove the header and connect again as above.

## Tools

The authoritative list is what the server answers to `tools/list`. As of version 1.1.0:

| Tool | What it does | Kind |
| --- | --- | --- |
| **Games** | | |
| `list_applications` | Lists your organization's games, newest first | read-only |
| `create_application` | Creates a new game as a draft | write |
| `get_application` | Reads a game's saved form, archives and media | read-only |
| `update_application_form` | Saves form fields; fields you leave out keep their values | write |
| `get_submission_state` | Tells whether the game can be submitted to moderation now, and why not | read-only |
| `list_moderation_comments` | Reads the moderation correspondence on a game | read-only |
| `get_launch_steps` | Reads a game's launch path in order — archive, Bridge SDK, covers, form, sandbox, share, traffic — with the current step and the tools that move each one | read-only |
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
| **Docs** | | |
| `get_bridge_sdk_docs` | Reads the live Playgama Bridge SDK wiki, the whole index or one page | read-only |
| **Sandbox** | | |
| `get_sandbox_state` | Reads what is live in the sandbox and whether a publish would be accepted | read-only |
| `get_sandbox_share` | Reads the ready post and share links for the live sandbox | read-only |
| `publish_sandbox` | Makes a build playable by anyone with the link, without moderation | write |
| `get_sandbox_traffic` | Reads whether traffic can be brought to the sandbox, the package on offer and the runs so far | read-only |
| `start_sandbox_traffic` | Starts a DSP campaign built from the game's covers that sends players to the sandbox; free traffic is a share bonus and takes 1–3 public post links | write |

Every write tool is annotated `destructiveHint: true`, so clients ask before calling it.

Agents start with `get_launch_steps` and read it again after each step.

### Uploading a build

1. `start_archive_upload` answers `uploadUrl` and `headers`.
2. PUT the zip to `uploadUrl` with exactly those headers, e.g.
   `curl -T game.zip -H "Content-Type: application/zip" "<uploadUrl>"`.
3. `confirm_archive_upload`, then poll `get_archive_status` until `processing` is `FAILED`
   (upload a corrected zip), or `processing` is `DONE` and `bridgeSdk` is no longer `PENDING`;
   if `bridgeSdk` is still `PENDING` after a few minutes, go ahead.
   `bridgeSdk: NOT_FOUND` means the Playgama Bridge SDK was not detected: it does not stop a
   sandbox publish, but sandbox traffic requires it — tell the developer before publishing;
   `get_bridge_sdk_docs` has the integration docs.

Covers work the same way, one slot per call: a PNG or JPEG of exactly 800×800 (square),
1080×1920 (portrait) or 1920×1080 (landscape).

### Sandbox traffic

Get a free boost after sharing your game on any platform. The developer publishes the post and
pastes its link to the agent, which passes one to three public HTTPS post links to `start_sandbox_traffic` in `postUrls`; one is enough. The bonus is once per game,
for up to three games per organization over its lifetime. Earlier free launches without sharing
do not use this bonus. `get_sandbox_share` has the ready post and share links. Paid traffic is
available in the cabinet; MCP does not purchase it.

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

## Beyond the MCP server

The rest of the Playgama stack picks up where this server stops:

- **[Playgama Bridge SDK](https://wiki.playgama.com/playgama/bridge-sdk/getting-started)** — one API
  for ads, leaderboards, payments and platform SDKs when you publish the same game across platforms.
  Open source, `npm i @playgama/bridge`.
- **[Playgama Ad](https://playgama.com/adv)** — monetizes web games with rewarded, interstitial and
  banner formats through a single lightweight JS SDK, with gaming-focused demand, Google Ad Manager
  MCM support and reporting. Access by request.
- **[Playgama Wrap](https://playgama.com/wrap/)** — turns the same game into a standalone site on your
  own domain, with hosting, player accounts, in-game purchases, PWA install and indexable SEO pages.
  Early access.
- **Playgama DSP** — brings the first players to a freshly published game. Beta.

## License

[MIT](LICENSE)
