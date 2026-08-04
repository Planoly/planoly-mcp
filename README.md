# Planoly MCP Server

[![MCP](https://img.shields.io/badge/MCP-remote%20server-blue)](https://modelcontextprotocol.io)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)
[![Docs](https://img.shields.io/badge/docs-tool%20reference-green)](./docs/tools.md)

Connect AI assistants to [Planoly](https://www.planoly.com) — plan, create, schedule, and analyze social media content across Instagram, TikTok, YouTube, Pinterest, Facebook, and more, straight from Claude, Cursor, or any MCP client.

This is a **remote MCP server**, hosted by Planoly:

```
https://mcp.planoly.com/mcp
```

There is nothing to install or run — you connect your MCP client to the URL above and sign in with your Planoly account via OAuth. This repository hosts the public documentation and registry metadata for the server.

## What you can do

- **Create and schedule posts** across all your connected channels in one call — shared caption/media with per-platform overrides, Instagram tags and collaborators, TikTok sounds and privacy settings, YouTube visibility, Pinterest boards, and more.
- **Manage your content calendar** — list drafts and scheduled posts, edit captions and settings, reschedule (including Instagram Best Time slots), or delete.
- **Work with your media library** — browse, search, organize into folders, preview media, import from URLs, or upload new assets.
- **Analyze performance** — Instagram account analytics, per-post performance ranked by the metric you care about, public account research via Business Discovery, and shareable report links.
- **Discover trends** — browse TikTok's Commercial Music Library trending sounds and attach them to posts.

See [docs/examples.md](./docs/examples.md) for example prompts and workflows.

## Getting started

### Claude Code

```bash
claude mcp add --transport http planoly https://mcp.planoly.com/mcp
```

### Claude (claude.ai and Claude Desktop)

Go to **Settings → Connectors → Add custom connector** and enter:

```
https://mcp.planoly.com/mcp
```

### Cursor

Add to `~/.cursor/mcp.json` (or `.cursor/mcp.json` in a project):

```json
{
  "mcpServers": {
    "planoly": {
      "url": "https://mcp.planoly.com/mcp"
    }
  }
}
```

### VS Code

```bash
code --add-mcp '{"name":"planoly","type":"http","url":"https://mcp.planoly.com/mcp"}'
```

### Other clients (stdio-only)

For clients that only support local stdio servers, bridge with [`mcp-remote`](https://www.npmjs.com/package/mcp-remote):

```json
{
  "mcpServers": {
    "planoly": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://mcp.planoly.com/mcp"]
    }
  }
}
```

On first use, your client opens a browser window to sign in to Planoly and approve access. If the bridge's auth state gets stuck, clear it with `rm -rf ~/.mcp-auth` and reconnect.

## Authentication

The server uses the standard MCP authorization flow — OAuth 2.1 authorization code with refresh tokens, discovered automatically by the client:

- Protected resource metadata: `https://mcp.planoly.com/.well-known/oauth-protected-resource/mcp`
- Authorization server: `https://app.planoly.com/api/auth`
- Scopes: `openid`, `profile`, `email`, `offline_access`

Tokens are scoped to your Planoly user. Tools that operate on a workspace take a `workspaceId` and enforce workspace membership on every call — start with `list_workspaces` to see what you can access.

Full details in [docs/authentication.md](./docs/authentication.md).

## Tools

Summarized below; parameter-level detail lives in **[docs/tools.md](./docs/tools.md)**.

### Workspaces & channels

| Tool | Description |
|------|-------------|
| `list_workspaces` | List the workspaces your account can access. |
| `list_channels` | List a workspace's connected social channels and their connection state, so the agent knows what it can post to. |

### Posts & scheduling

| Tool | Description |
|------|-------------|
| `create_post` | Create one post group across one or more channels. Targets share a caption/media set or override per platform, with platform settings for Instagram (tags, cover, collaborators, first comment, sound), TikTok (title, privacy, branded content, music), YouTube (title, visibility), Pinterest (board, title, link), Facebook, and Amazon. Optionally schedule the whole group. |
| `list_post_groups` | List existing posts (drafts, scheduled, published) with captions, schedule dates, media (including reusable asset IDs), and thumbnails. |
| `update_post` | Edit an existing post — caption, media, first comment, and per-platform settings. |
| `set_post_group_schedule` | Schedule a draft, reschedule, schedule into the next Instagram Best Time slot, or revert to draft. Auto-publishing posts are validated against platform rules before scheduling. |
| `delete_post_group` | Delete a draft or scheduled post group. |

### Media library & assets

| Tool | Description |
|------|-------------|
| `list_media_library` | Browse and search the media library — photos, videos, carousels, notes, URL bookmarks, and folders. |
| `get_media_preview` | Return an inline thumbnail image for an asset so any MCP client can see what's in a photo or video. |
| `create_media_library_folder` | Create a folder (optionally nested) to organize library items. |
| `create_media_library_item` | Save media, a note, or a URL bookmark into the library. |
| `update_media_library_item` | Edit a library entry's name, title, description, link, or media set. |
| `delete_media_library_item` | Remove a library entry (folders cascade; underlying assets and posts are untouched). |
| `create_asset_from_url` | Import an image/video from a URL on the supported-host allowlist (Instagram/Facebook CDNs, Planoly, Canva exports, and supported AI-generation outputs). |
| `request_asset_upload` / `confirm_asset_upload` | Signed-URL upload flow for clients that can perform an HTTP PUT (e.g. a code-execution sandbox). |

### Analytics & research

| Tool | Description |
|------|-------------|
| `get_instagram_analytics` | Instagram Business account analytics for a date range — follower growth, reach, views, interactions, audience breakdowns, and a daily series. |
| `list_post_performance` | How published Instagram posts performed — reach, views, likes, comments, shares, saves — ranked by a chosen metric. |
| `get_instagram_account_posts` | Fetch a public Instagram Business/Creator account's recent posts (captions, media, public engagement) for research and digests. |
| `create_post_report_share_link` | Mint a shareable, read-only link to the workspace's post performance report (expires in 14 days). |

### Instagram tagging & TikTok sounds

| Tool | Description |
|------|-------------|
| `search_instagram_locations` | Search places to tag in Instagram posts. |
| `search_instagram_products` | Search a connected Instagram Shop catalog for product tagging. |
| `list_tiktok_trending_sounds` | Browse TikTok Commercial Music Library trending sounds by genre, country, and date range. |

## How media gets into posts

The model never carries media bytes through inference. Media reaches a post by `assetId`, from one of three sources:

1. **Existing media** — take an `assetId` from `list_media_library` or from another post via `list_post_groups`.
2. **From a URL** — `create_asset_from_url` has the server fetch it (allowlisted hosts only).
3. **Local bytes** — `request_asset_upload` → HTTP `PUT` the bytes → `confirm_asset_upload`, for clients with a code-execution sandbox.

Details and constraints in [docs/media.md](./docs/media.md).

## Validation & plan requirements

Posts are validated against each platform's rules (caption length, media count/type, aspect ratio, video duration) the same way the Planoly app validates them — errors come back at tool-call time with the exact violation, not at publish time.

Some tools map to Planoly premium features and require a paid workspace: TikTok trending sounds, the media library tools, the analytics tools, and product tagging. When a tool requires an upgrade, the error includes a link to the billing page.

## Documentation

- [Tool reference](./docs/tools.md) — every tool, its parameters, and behavior
- [Authentication](./docs/authentication.md) — the OAuth flow, token model, resetting auth state
- [Media handling](./docs/media.md) — how assets get in and out
- [Example prompts](./docs/examples.md) — workflows to try
- [Troubleshooting](./docs/troubleshooting.md) — common errors and fixes
- [Changelog](./CHANGELOG.md)

## Support & security

- Product and account help: [Planoly help center](https://support.planoly.com) · see [SUPPORT.md](./SUPPORT.md)
- MCP server bugs and feature requests: [issues](https://github.com/Planoly/planoly-mcp/issues)
- Vulnerabilities: report privately per [SECURITY.md](./SECURITY.md)

The server implementation is not open source; this repository tracks the public documentation and [MCP registry](https://registry.modelcontextprotocol.io) metadata ([server.json](./server.json)) for the hosted service.
