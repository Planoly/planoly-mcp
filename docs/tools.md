# Tool reference

All tools that operate on workspace data take a `workspaceId`. Call `list_workspaces` first to see which workspaces your account can access, and `list_channels` to see what's connected and postable.

- [Workspaces & channels](#workspaces--channels)
- [Posts & scheduling](#posts--scheduling)
- [Media library & assets](#media-library--assets)
- [Analytics & research](#analytics--research)
- [Instagram tagging & TikTok sounds](#instagram-tagging--tiktok-sounds)

## Workspaces & channels

### `list_workspaces`

Lists the workspaces the authenticated user can access. Returns each workspace's `id`, `name`, and your `role` in it.

### `list_channels`

Lists a workspace's social channels: `platform`, `username`, `connectionState`, and `isReminderOnly`. Use it to know which platforms a post can target and whether a channel auto-publishes or sends a reminder notification instead. Never returns tokens or secrets.

## Posts & scheduling

### `create_post`

The general create tool — builds one **post group** across one or more connected channels.

- `targets[]` lists the platforms. Each target carries its own `postType` and optional per-platform `caption`/`media` overrides. A target inherits the shared top-level `caption`/`media` unless it overrides them — so cross-posting the same content is one target list, and tailoring per platform is per-target overrides.
- Per-platform settings: Instagram (tags, cover, collaborators, first comment, creator-store link, sound), TikTok (title, privacy, branded content, disable comments, auto music, photo cover, music), YouTube (title, visibility, notify subscribers), Pinterest (board, title, link), Facebook (link), Amazon (products).
- Instagram targets support `tags` (user / location / product, with x/y position) and a custom `coverAssetId` for reels. Location ids come from `search_instagram_locations`; product ids from `search_instagram_products`.
- TikTok `music` takes a Commercial Music Library sound — pick one from `list_tiktok_trending_sounds` and pass its `commercialMusicId`.
- `scheduleDate` schedules the whole group; omit it to save a draft.
- Media items are passed by `assetId` — see [media.md](./media.md) for how assets get created.

Validation: drafts save freely. When a post is created **scheduled to auto-publish**, it is validated against the target platform's content rules first (see [Validation](#validation) below). Caption length is the exception — it is enforced on every create/update regardless of status, per platform, with the exact overage in the error.

### `list_post_groups`

Lists existing posts (drafts, scheduled, published) so the agent can find ones to edit, reschedule, or delete. Returns `postGroupId` (for schedule/delete), each post's `postId` (for `update_post`), status, `scheduleDate`, caption, `firstComment`, `collaborators`, and `media` — each item's `assetId`, `type`, `mimeType`, CDN `url` and `thumbnailUrl`, plus any post-specific tags or cover. The asset ids let you **reuse an existing post's media** in a new post.

### `update_post`

Edits an existing post by `postId` — caption, media, first comment, auto-publish flag, and the same per-platform settings `create_post` accepts. Only the fields passed change; platform settings merge over the post's existing ones. Timing is not changed here — use `set_post_group_schedule`.

### `set_post_group_schedule`

Sets or clears a post group's schedule by `postGroupId`:

- `scheduleDate` — schedule a draft or reschedule an existing post group.
- `useBestTime` — schedule into the next available Instagram **Best Time** slot instead of an explicit date (requires Best Time enabled for a connected Instagram business/creator account with a timezone set). Mutually exclusive with `scheduleDate`.
- Omit both — revert the group to a draft (cancels the scheduled publish, keeps content).

This is the hard validation gate: when scheduling, each auto-publishing post is checked against its platform's rules first, so a draft that would fail at publish time is rejected here with a fixable error instead.

### `delete_post_group`

Deletes a post group (draft or scheduled) and its posts.

### Validation

Platform posting rules — caption length, media count/type, aspect ratio, video duration, file size, first comment, collaborators, location — are enforced the same way the Planoly app enforces them:

- Drafts (including drafts holding a future `scheduleDate`) and reminder-only posts save without content gating.
- Scheduling an auto-publish post is validated up front; violations come back as a tool error listing every problem.
- Caption length is checked on **every** create/edit, per platform — a shared caption that fits Instagram (2,200) but exceeds another platform's limit is caught for that platform. Caption-less post types (e.g. stories) are exempt.

## Media library & assets

### `list_media_library`

Browses the media library. `type` filters to `media` (default — photos/videos and carousels), `note`, `url` bookmarks, or `folder`. `parentId` scopes the listing (a folder id browses inside it, `null` is the workspace root, omitted spans all folders). Supports `search` over title/description. Returns each item's `assets[]` with `assetId`, `type`, `mimeType`, CDN `url`, and `thumbnailUrl`.

### `get_media_preview`

Returns an actual inline thumbnail **image** (one asset per call) so any MCP client can see what's in a photo, carousel slide, or video (videos return their cover frame). URLs in other tool results are text-only; this tool inlines the pixels.

### `create_media_library_folder`

Creates a folder, optionally nested under `parentId` (max 10 levels). Returns the folder `libraryId`.

### `create_media_library_item`

Saves an item into the library for reuse: `media` (one `assetId`, or several as a carousel), a `note`, or a `url` bookmark, optionally placed in a folder. To attach media straight to a post instead, pass it directly as `media` entries in `create_post`.

### `update_media_library_item`

Edits an existing entry — a folder's `name`, or an item's title, description, bookmark link, or media asset set (full replacement). Does not move entries between folders.

### `delete_media_library_item`

Removes a library entry. Deleting a **folder cascades** to everything inside it. Underlying assets and posts using them are untouched.

### `create_asset_from_url`

Imports an image/video from a fetchable URL — the server downloads it. The source host must be on the supported allowlist: Instagram/Facebook CDNs, Planoly hosts, supported AI-generation output hosts, and design-tool export hosts (e.g. Canva). Intended for re-importing social/Planoly media, AI-generated output, and design exports — not arbitrary URLs.

### `request_asset_upload` / `confirm_asset_upload`

Signed-URL upload for clients that can perform an HTTP `PUT` (a code-execution sandbox such as Claude Code or the claude.ai sandbox). `request_asset_upload` returns an `assetId` + short-lived `uploadUrl`; the client PUTs the raw bytes; `confirm_asset_upload` verifies and marks the asset ready. Not usable from a plain chat turn with no sandbox — use `create_asset_from_url` or existing library media instead.

## Analytics & research

### `get_instagram_analytics`

Instagram Business **account** analytics for a date range: followers and net growth, reach/views/interactions with previous-period comparison, interactions by content type (feed/reels/story), new-vs-returning audience, top demographics, and a daily reach series. Live from Instagram (cached ~3h). Instagram business accounts only.

### `list_post_performance`

How the workspace's **published** Instagram posts performed over a range — per-post reach, views, likes, comments, shares, saves (plus reel watch time), ranked by a metric you choose. Includes thumbnails where available, and reports `total` + `hasMore` for paging.

### `get_instagram_account_posts`

Fetches a **public** Instagram Business/Creator account's recent posts via Business Discovery — for competitor and creator research. Returns caption, media type/URLs, permalink, public engagement counts, and timestamps, paginated via `nextCursor` (default 32, max 50). Only public counts are available (not private insights). Requires the workspace to have its own connected Instagram Business account; the target only needs to be a public business/creator handle.

### `create_post_report_share_link`

Mints a shareable, read-only link to the workspace's post-performance report — viewable as a public web page without a login. The link expires in 14 days.

## Instagram tagging & TikTok sounds

### `search_instagram_locations`

Searches Instagram/Facebook places to tag. Returns `id` (use as a `location` media-tag value), name, and address. Resolves through the workspace's connected Instagram business/creator account.

### `search_instagram_products`

Searches a connected Instagram Shop catalog. Returns `productId` (use as a `product` media-tag value), name, image, and catalog; when the account has several catalogs, the catalog list is returned to choose from.

### `list_tiktok_trending_sounds`

Browses TikTok's Commercial Music Library trending list (TikTok has no keyword sound search), filterable by `genre`, `countryCode`, and `dateRange`. Returns each sound's `commercialMusicId`, name, artist, duration, and preview. Pass the id into a TikTok target's `music` on `create_post`. Requires a connected TikTok Business account.
