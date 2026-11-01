# Planoly MCP Server

[![MCP](https://img.shields.io/badge/MCP-remote%20server-blue)](https://modelcontextprotocol.io)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)
[![Docs](https://img.shields.io/badge/docs-tool%20reference-green)](./docs/tools.md)

Connect AI assistants to [Planoly](https://www.planoly.com) — plan, create, schedule, and analyze social media content across Instagram, TikTok, YouTube, Pinterest, Facebook, LinkedIn, Threads, and X, straight from Claude, Cursor, or any MCP client.

This is a **remote MCP server**, hosted by Planoly:

```
https://mcp.planoly.com/mcp
```

There is nothing to install or run — you connect your MCP client to the URL above and sign in with your Planoly account via OAuth. This repository hosts the public documentation and registry metadata for the server.

## What you can do

- **Create and schedule posts** across all your connected channels in one call — Instagram, TikTok, YouTube, Pinterest, Facebook, LinkedIn, Threads, and X — with shared caption/media and per-platform overrides: Instagram tags, collaborators and reel audio, TikTok sounds and privacy, YouTube visibility, Pinterest boards, and more.
- **Manage your content calendar** — list drafts and scheduled posts, see why a post failed, edit captions and settings, reschedule (including Instagram Best Time slots), or delete.
- **Work with your media library** — browse, search, organize and move between folders, preview media, import from URLs, or upload new assets.
- **Analyze performance** — Instagram, TikTok, LinkedIn, and Pinterest analytics, per-post Instagram performance, public account research, and shareable report links.
- **Write in your brand voice** — have Planoly learn how you write from your own posts, then read it back before writing captions.
- **Handle your inbox** — read and search Instagram DMs and comments, reply (with your saved replies), and triage with labels and assignments.
- **Run Instagram automations** — auto-DM people who comment a keyword, reply to a story, or mention you.
- **Get answers** — search Planoly's help center and check your plan and billing.

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

52 tools, summarized below; parameter-level detail lives in **[docs/tools.md](./docs/tools.md)**.

### Help, workspaces & billing

| Tool | Description |
|------|-------------|
| `search_help` / `get_help_article` | Search and read Planoly's help center. |
| `list_workspaces` | List the workspaces your account can access, and whether each includes the inbox. |
| `list_workspace_members` | List the people in a workspace. |
| `get_account_billing` | Your plan, renewal, trial, limits, and recent payments (read-only). |

### Channels

| Tool | Description |
|------|-------------|
| `list_channels` | List a workspace's connected channels, whether they work, and whether they auto-publish. |
| `list_pinterest_boards` | List the boards on the connected Pinterest account. |

### Posts & scheduling

| Tool | Description |
|------|-------------|
| `create_post` | Create one post group across one or more channels, with shared or per-platform caption/media and platform settings. Optionally schedule it. |
| `list_post_groups` | List posts (drafts, scheduled, published, failed) with captions, schedule, media, and why a post failed. |
| `update_post` | Edit a post — caption, media, first comment, format, and per-platform settings. |
| `set_post_group_schedule` | Schedule, reschedule, schedule at the next Instagram Best Time slot, or revert to draft. |
| `delete_post_group` | Delete a draft or scheduled post group. |

### Instagram tagging & sounds

| Tool | Description |
|------|-------------|
| `search_instagram_locations` | Search places to tag. |
| `search_instagram_products` | Search a connected Instagram Shop catalog for product tags. |
| `search_instagram_audio` | Search or browse Instagram's audio catalog for reel sounds. |
| `list_tiktok_trending_sounds` | Browse TikTok Commercial Music Library trending sounds. |

### Media library & assets

| Tool | Description |
|------|-------------|
| `list_media_library` | Browse and search photos, videos, carousels, notes, bookmarks, and folders. |
| `get_media_preview` | Return an inline thumbnail so any client can see an asset. |
| `create_media_library_folder` | Create a folder (optionally nested). |
| `create_media_library_item` | Save media, a note, or a bookmark into the library. |
| `update_media_library_item` | Edit an entry or move it to another folder. |
| `delete_media_library_item` | Remove an entry (folders cascade; assets and posts are untouched). |
| `create_asset_from_url` | Import an image/video from a supported source (Instagram/Facebook, Planoly, Canva, Higgsfield, DALL·E). |
| `request_asset_upload` / `confirm_asset_upload` | Signed-URL upload for clients that can run code. |

### Analytics & research

| Tool | Description |
|------|-------------|
| `get_instagram_analytics` | Instagram account analytics — growth, reach, interactions, audience. |
| `list_post_performance` | Published Instagram posts ranked by the metric you choose. |
| `get_tiktok_analytics` | TikTok account analytics, demographics, and recent posts. |
| `get_linkedin_analytics` | LinkedIn page or profile analytics and recent posts. |
| `get_pinterest_analytics` | Pinterest analytics, top pins, and recent pins. |
| `get_instagram_account_posts` | A public Instagram Business/Creator account's recent posts, for research. |
| `create_post_report_share_link` | A shareable, read-only post-performance report link (expires in 14 days). |

### Brand voice

| Tool | Description |
|------|-------------|
| `get_brand_voice` | Read how the brand writes, plus its look and strategy. |
| `request_brand_voice_build` | Learn the brand voice from the user's own posts (read-only; the user approves the result). |
| `save_brand_voice` | Approve, restore, or add notes, look, or strategy to the brand voice. |
| `forget_brand_voice` | Delete the brand voice (restorable). |

### Inbox

| Tool | Description |
|------|-------------|
| `list_inbox_conversations` / `search_inbox_conversations` | List or search Instagram DMs and comment threads. |
| `get_inbox_conversation` | Read one conversation. |
| `send_inbox_reply` | Reply by DM, public comment, or private reply. |
| `update_inbox_conversation` | Mark read/unread, close/reopen, assign, and label conversations in bulk. |
| `list_saved_replies` | List saved replies to send again. |
| `list_inbox_labels` / `create_inbox_label` / `update_inbox_label` / `delete_inbox_label` | Manage inbox labels. |

### Instagram automations

| Tool | Description |
|------|-------------|
| `list_automations` / `get_automation` | List or read keyword, story-reply, DM, and mention auto-replies. |
| `create_automation` / `update_automation` | Create (saved paused) or change an automation. |
| `set_automation_enabled` | Turn an automation on or pause it. |
| `delete_automation` | Delete an automation permanently. |

## How media gets into posts

The model never carries media bytes through inference. Media reaches a post by `assetId`, from one of three sources:

1. **Existing media** — take an `assetId` from `list_media_library` or from another post via `list_post_groups`.
2. **From a URL** — `create_asset_from_url` has the server fetch it (allowlisted hosts only).
3. **Local bytes** — `request_asset_upload` → HTTP `PUT` the bytes → `confirm_asset_upload`, for clients with a code-execution sandbox.

Details and constraints in [docs/media.md](./docs/media.md).

## Validation & plan requirements

Posts are validated against each platform's rules (caption length, media count/type, aspect ratio, video duration) the same way the Planoly app validates them — errors come back at tool-call time with the exact violation, not at publish time.

Some tools map to Planoly premium features and require a paid workspace — including the media library, analytics, TikTok trending sounds, product tagging, and the inbox. When a tool requires an upgrade, the error includes a link to the pricing page. On the free plan, new posts count against a monthly post allowance.

Tools that send to real people — `send_inbox_reply` and turning on an automation — act on Instagram immediately. Their descriptions tell the assistant to confirm the exact action with you first.

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
