# Changelog

Notable changes to the hosted Planoly MCP server (`https://mcp.planoly.com/mcp`) and this documentation.

## 2026-11-01

- 29 new tools — 52 in total. See [docs/tools.md](./docs/tools.md).
  - **Help:** `search_help`, `get_help_article`.
  - **Workspaces & billing:** `list_workspace_members`, `get_account_billing`.
  - **Channels:** `list_pinterest_boards`.
  - **Posts:** `search_instagram_audio` for reel sounds.
  - **Analytics:** `get_tiktok_analytics`, `get_linkedin_analytics`, `get_pinterest_analytics`.
  - **Brand voice:** `get_brand_voice`, `request_brand_voice_build`, `save_brand_voice`, `forget_brand_voice`.
  - **Inbox:** `list_inbox_conversations`, `search_inbox_conversations`, `get_inbox_conversation`, `send_inbox_reply`, `update_inbox_conversation`, `list_saved_replies`, `list_inbox_labels`, `create_inbox_label`, `update_inbox_label`, `delete_inbox_label`.
  - **Instagram automations:** `list_automations`, `get_automation`, `create_automation`, `update_automation`, `set_automation_enabled`, `delete_automation`.
- `create_post` / `update_post`: post formats for Facebook, X, LinkedIn, and Threads; Instagram `excludeFromGrid`, `audio`, and `trialGraduation`; TikTok creator-store link. Results report `willAutoPublish` and a `connectUrl` when reconnecting would enable auto-publishing. Amazon is no longer a target for new posts.
- `list_post_groups`: paging with `cursor`, plus `latestError`, publish warnings, and the live post URL.
- `list_channels`: reports whether each channel can auto-publish, login refresh failures, and Instagram messaging problems.
- `update_media_library_item`: can move entries between folders.
- `create_asset_from_url`: supported sources are now Instagram/Facebook, Planoly media and stock library, Canva, Higgsfield, and DALL·E.
- New error codes: `POST_GROUP_CREATION_LIMIT_REACHED` (free plan monthly allowance), `RATE_LIMIT_REACHED`, `PROVIDER_ERROR`.

## 2026-08-04

- Initial public documentation release.
- Server surface: 23 tools across posting & scheduling, media library & assets, analytics & research, Instagram tagging, and TikTok sounds. See [docs/tools.md](./docs/tools.md).
- Authentication: OAuth 2.1 authorization code + refresh via `https://app.planoly.com/api/auth`, standard MCP discovery.
- Published MCP registry metadata ([server.json](./server.json)).
