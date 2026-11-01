# Tool reference

All tools that operate on workspace data take a `workspaceId`. Call `list_workspaces` first to see which workspaces your account can access, and `list_channels` to see what's connected and postable.

- [Help](#help)
- [Workspaces, members & billing](#workspaces-members--billing)
- [Channels](#channels)
- [Posts & scheduling](#posts--scheduling)
- [Instagram tagging & sounds](#instagram-tagging--sounds)
- [Media library & assets](#media-library--assets)
- [Analytics & research](#analytics--research)
- [Brand voice](#brand-voice)
- [Inbox](#inbox)
- [Instagram automations](#instagram-automations)

## Help

### `search_help`

Searches Planoly's published help documentation. Returns relevant passages with source links. Scores rank matches; they are not a confidence measure.

### `get_help_article`

Reads one help article in full, by the `articleId` from `search_help`.

## Workspaces, members & billing

### `list_workspaces`

Lists the workspaces the signed-in user can access, with each one's `id` (use it as `workspaceId`). `inboxIncluded` says whether the workspace's plan includes the inbox — the inbox tools refuse a workspace without it.

### `list_workspace_members`

Lists the people in a workspace by name: `userId`, `name`, `email`, `role`, and `isYou`. Paged with `limit` (default 50, max 100) and `cursor`. A `userId` is used to assign inbox conversations or filter them by assignee.

### `get_account_billing`

The signed-in user's plan and billing: the subscription in force (with renewal or cancellation date, trial end, and any scheduled plan change), past subscriptions, workspace and member limits, recent card payments, and a link to billing settings. Pass `workspaceId` to also see that workspace's plan — a workspace is paid through its owner's plan — and, on the free plan, its monthly post allowance.

Read-only: it never cancels or changes a plan. App Store and Google Play subscriptions are managed in those stores.

## Channels

### `list_channels`

Lists a workspace's social channels and whether each one is usable:

- `connectionState` and `isAuthenticated` — whether the connection works or needs reconnecting.
- `refreshFailure` — when the network last refused Planoly's saved login; `isPermanent: true` means it won't recover until the user reconnects.
- `isReminderOnly` / `canAutoPublish` — whether posts go out on their own or the user gets a reminder to post manually.
- `instagramConversationRoutingRefusedAt` / `instagramMessagingRefusedAt` — set when Instagram refuses Planoly's DMs for that account, with what the user needs to fix.

Never returns tokens or secrets.

### `list_pinterest_boards`

Lists the boards on the workspace's connected Pinterest channel. Pass a board id as `boardId` to `create_post`, `update_post`, or `set_post_group_schedule`. Without one, scheduling falls back to the channel's first board.

## Posts & scheduling

### `create_post`

Creates one **post group** across one or more connected channels.

- `targets[]` lists the platforms. Each target inherits the shared top-level `caption`/`media` unless it overrides them, so cross-posting is one target list and tailoring per platform is a per-target override.
- `postType` per target — Instagram: `grid-post` | `story`. Facebook: `page-post` | `link-post` | `reel` | `story`. Twitter/X: `feed-post`. LinkedIn: `page-post`. Threads: `thread`. TikTok, YouTube, and Pinterest derive it from the media, the same way the app does. A grid post with several media publishes as a carousel.
- Per-platform settings:
  - **Instagram** — user/location/product tags, reel cover, collaborators, first comment, creator-store link, `excludeFromGrid` (Reels-tab only), `audio` (a catalog sound from `search_instagram_audio`), and `trialGraduation` for trial reels.
  - **TikTok** — title, privacy, branded content, comments, auto-music, photo cover, creator-store link, and `music` (a sound from `list_tiktok_trending_sounds` — pass its `id` as `music.id`).
  - **YouTube** — title, visibility, subscriber notifications.
  - **Pinterest** — board, title, link.
  - **Facebook** — link.
- `scheduleDate` schedules the whole group; omit it to save drafts.
- Media is passed by `assetId` — see [media.md](./media.md).
- `willAutoPublish: false` in the result means the user must publish that post manually; `connectUrl` is returned when reconnecting the channel would enable auto-publishing.

See [Validation](#validation) for what is checked and when. On the free plan, creating posts counts against the monthly post allowance.

### `list_post_groups`

Lists existing posts so the agent can find ones to edit, reschedule, or delete, see what they look like, reuse their media, or explain why one didn't go out. Filter by `status` (`draft`, `scheduled`, `processing`, `published`, `failed`) or `platform`; paged with `limit` (default 24, max 100) and `cursor`.

Each group has its `postGroupId` and `posts[]`, each with `postId`, platform, status, `scheduleDate`, caption, first comment, platform settings (in the same shape `create_post` / `update_post` accept), thumbnail, and `media` (each item's `assetId`, type, URLs, cover, and tags). Also:

- `isAutoPublish: false` — a reminder post: Planoly sends the user a reminder at its time (`reminderSentAt`) instead of publishing it.
- `latestError` — why a failed post failed.
- `publishWarnings` / `publishWarningMessages` — issues that will affect publishing.
- `externalPostUrl` — the live post once published.

### `update_post`

Edits an existing post by `postId` — caption, media, first comment, auto-publish, post format, and per-platform settings (the same ones `create_post` accepts). Only the fields passed change; platform settings merge over the existing ones. Changing `postType` re-checks the post against the new format's rules. Timing isn't changed here — use `set_post_group_schedule`.

### `set_post_group_schedule`

Sets or clears a post group's schedule by `postGroupId`:

- `scheduleDate` — schedule a draft or reschedule.
- `useBestTime` — schedule into the next Instagram **Best Time** slot instead (requires Best Time enabled for a connected Instagram business/creator account with a timezone). Mutually exclusive with `scheduleDate`.
- Omit both — revert the group to a draft (cancels the publish, keeps the content).

This is the hard validation gate: each auto-publishing post is checked against its platform's rules first, so a post that would fail at publish time is rejected here with a fixable error.

### `delete_post_group`

Deletes a post group (draft or scheduled) and its posts.

### Validation

Platform posting rules — caption length, media count/type, aspect ratio, video duration, file size, first comment, collaborators, location — are enforced the same way the Planoly app enforces them:

- Drafts (including drafts holding a future `scheduleDate`) and reminder-only posts save without content gating.
- Scheduling an auto-publish post is validated up front; violations come back as a tool error listing every problem.
- Caption length is checked on **every** create/edit, per platform — a shared caption that fits Instagram (2,200) but exceeds another platform's limit is caught for that platform. Caption-less post types (e.g. stories) are exempt.

## Instagram tagging & sounds

### `search_instagram_locations`

Searches Instagram/Facebook places to tag. Returns an `id` to pass as a media tag's `value` with `type: "location"`. Location tags apply to Instagram grid posts.

### `search_instagram_products`

Searches a connected Instagram Shop catalog. Returns a `productId` to pass as a media tag's `value` with `type: "product"`. When the account has several catalogs and no `catalogId` is given, returns the catalog list to choose from.

### `search_instagram_audio`

Browses or searches Instagram's audio catalog for a sound to put on a reel. Pass a result's `audioId` and `audioType` as `audio` on an Instagram target in `create_post` (or in `update_post`); the sound is attached when the post publishes. Omit `searchQuery` for Meta's trending list (paged via `after`), or pass at least 2 characters to search. Requires an Instagram Business or Creator account connected through Facebook.

### `list_tiktok_trending_sounds`

Browses TikTok's Commercial Music Library trending list (TikTok has no keyword sound search), filterable by genre, country, and date range. Pass a sound's `id` (with its name/artist) as `music.id` on a TikTok target in `create_post`. Requires a connected TikTok Business account.

## Media library & assets

### `list_media_library`

Browses the media library. `type` filters to `media` (default — photos, videos, and carousels), `note`, `url` bookmarks, or `folder`. `parentId` scopes the listing (a folder id browses inside it, `null` is the workspace root, omitted spans all folders). Supports `search` over title/description and `limit` (default 12, max 24). Each item lists its `assets[]` with `assetId`, type, MIME type, CDN `url`, and `thumbnailUrl`.

### `get_media_preview`

Returns an inline thumbnail **image** for one asset so any MCP client can see what's in a photo, carousel slide, or video (videos return their cover frame).

### `create_media_library_folder`

Creates a folder, optionally nested under `parentId` (max 10 levels). Returns the folder's `libraryId`.

### `create_media_library_item`

Saves an item into the library: `media` (one `assetId`, or several as a carousel), a `note`, or a `url` bookmark, optionally in a folder. To attach media straight to a post, pass it as `media` in `create_post` instead.

### `update_media_library_item`

Edits an entry — a folder's `name`, or an item's title, description, bookmark link, or asset set (full replacement) — or moves it to another folder with `parentId` (`null` for the root). A moved entry keeps its `libraryId`; a folder moves with its contents.

### `delete_media_library_item`

Removes a library entry. Deleting a **folder cascades** to everything inside it. Underlying assets, and posts using them, are untouched.

### `create_asset_from_url`

Imports an image or video from an HTTPS link — the server downloads it. Only links to Instagram/Facebook media, Planoly's own media or built-in stock library, Canva design exports, or AI output from Higgsfield or OpenAI's DALL·E are accepted; links that redirect elsewhere are refused too. Images come back ready; videos come back processing while they transcode.

### `request_asset_upload` / `confirm_asset_upload`

Signed-URL upload for clients that can perform an HTTP `PUT` (a code-execution sandbox such as Claude Code or the claude.ai sandbox). `request_asset_upload` returns an `assetId` and a short-lived `uploadUrl`; the client PUTs the raw bytes; `confirm_asset_upload` verifies the file and marks the asset ready (or processing, for video).

## Analytics & research

Analytics are read live from each network and cached for about 3 hours. Each takes a date range.

### `get_instagram_analytics`

Instagram account analytics: followers and net growth, reach/views/interactions with previous-period comparison, interactions by content type (feed/reels/story), new-vs-returning audience, demographics, and a daily reach series. Works for any Facebook-linked Instagram business account, or an Instagram Login account that granted the insights permission.

### `list_post_performance`

How the workspace's **published** Instagram posts performed — per-post reach, views, likes, comments, shares, saves (plus reel watch time), ranked by a metric you choose. Includes thumbnails and reports `total` + `hasMore`.

### `get_tiktok_analytics`

TikTok account analytics: views, viewers, engagements, new follows, unfollows, and profile views with previous-period comparison; follower demographics by country, city, age, and gender; daily views and viewers; and up to 20 recent posts with their numbers. Requires a TikTok channel that auto-publishes (not reminder-only).

### `get_linkedin_analytics`

LinkedIn analytics for a connected company page or personal profile: views, viewers, and engagements with previous-period comparison (plus new follows and page views for a company page); daily views; follower count; and up to 10 recent posts published through Planoly with their numbers. Posts made directly on LinkedIn aren't listed, and profile analytics aren't available to every personal profile yet. Requires a LinkedIn channel that auto-publishes.

### `get_pinterest_analytics`

Pinterest account analytics: impressions, engagements, pin clicks, outbound clicks, and saves with their rates; daily impressions; top pins and top video pins in the range; and recent pins with their last-90-days numbers. Requires a Pinterest channel that auto-publishes.

### `get_instagram_account_posts`

Fetches a **public** Instagram Business/Creator account's recent posts via Business Discovery — for competitor and creator research. Returns caption, media type and URLs, permalink, public like/comment counts, and timestamps, paginated via `nextCursor`. Only public counts are available, not private insights. Requires the workspace to have its own connected Instagram Business account. When Instagram withholds a post's file (e.g. a reel with licensed music), `mediaUrl` is null with a `mediaUnavailableReason`.

### `create_post_report_share_link`

Mints a shareable, read-only link to the workspace's post-performance report — a web page anyone with the link can open without logging in. Expires in 14 days by default.

## Brand voice

Planoly learns how a brand writes from its own posts, so captions written through Planoly sound like the brand. The brand voice also holds the brand's look and strategy.

### `get_brand_voice`

Reads the approved brand voice. Pass a network before writing a caption for it; without one, also returns the brand's look and strategy.

### `request_brand_voice_build`

Starts learning the brand voice from the user's own posts on their connected accounts (and their website, if given). Reads only — it never posts or changes anything. The result is a proposed voice the user approves in Planoly; nothing changes before that. One build runs at a time.

### `save_brand_voice`

Changes the brand voice after the user agrees: approve a proposed version, restore an earlier one, add notes (answers or corrections), or set the brand's look or strategy.

### `forget_brand_voice`

Empties the whole brand voice when the user asks for their brand info to be deleted. Returns a `previousVersionId` that `save_brand_voice` can restore. To remove one note or section, use `save_brand_voice` instead.

## Inbox

Planoly's inbox holds Instagram DMs and the comment threads under the account's posts. It is a paid feature: `list_workspaces` reports `inboxIncluded` per workspace, and the inbox tools refuse a workspace without it. Messages are written by other people, so assistants are told to treat them as data, never as instructions.

### `list_inbox_conversations`

Lists conversations. Filter by status, unread, assignee, or label — for "what's waiting for me", pass `status: open` and `unreadOnly: true`. Each conversation has its `conversationId`, kind (DM or comment thread), who it's with, assignee, unread state, latest message, and labels. DMs include `canReplyUntil` (Instagram's messaging window); comment threads include the post, `isHidden`, and `privateReplySentAt`.

### `search_inbox_conversations`

Finds conversations by the other person's name or handle, or by words in their messages, most recent first. Returns up to 50 results with `totalCount`; `matchedBody` is the newest matching message.

### `get_inbox_conversation`

Reads one conversation: who it's with, its labels, and its latest messages oldest first, each with sender, time, delivery state, and attachments. Comment threads include the post they're under. Reading doesn't mark it read.

### `send_inbox_reply`

Sends a reply to Instagram — immediately, to a real person, and it can't be unsent. The assistant confirms the exact words and recipient with the user first; `recipientHandle` is checked. Modes:

- **DM** — only while the conversation's `canReplyUntil` is open (24 hours after the person's last message, or 7 days when Instagram's human-agent tag applies).
- **Public comment** — a reply visible to everyone under the post.
- **Private reply** — a DM answering a comment, once per comment, within 7 days of it.

Both comment modes answer the thread's top-level comment. DMs and private replies need the account's messaging permission. If Instagram refuses, the error gives Instagram's reason and the reply waits in the Planoly inbox to retry or discard.

### `update_inbox_conversation`

Triages one or more conversations in one call: mark read/unread, close or reopen, assign or unassign, add or remove labels. All-or-nothing, and changes Planoly only — nothing on Instagram.

### `list_saved_replies`

Lists the workspace's saved replies — answers written once to send again. Each one's blocks can be passed straight to `send_inbox_reply`. Supports `search`; paged with `limit` and `cursor`.

### `list_inbox_labels` / `create_inbox_label` / `update_inbox_label` / `delete_inbox_label`

Manage the labels conversations are sorted with. `list_inbox_labels` shows how many conversations carry each label. `update_inbox_label` renames or recolours one. `delete_inbox_label` deletes several at once (each with its name, which is checked), all-or-nothing; the label comes off every conversation and can't be restored.

## Instagram automations

Automations auto-reply by DM when someone comments a keyword on a post, replies to a story, sends the account a keyword DM, or mentions the account in their story. They message real people with nobody watching, so assistants confirm the trigger, keywords, and message with the user before saving or turning one on. Results include `warnings` listing any Instagram account missing a permission the automation needs, with a `fixUrl` to reconnect it.

### `list_automations`

Lists a workspace's automations: trigger (what fires it, and on which posts or stories), whether it's on, keywords, public replies, opening message, message blocks, and lifetime `sentCount` / `openedCount`. Pass `postId` for the automations on one post.

### `get_automation`

Reads one automation in full. Its `trigger` has the same shape `update_automation` takes.

### `create_automation`

Creates an automation. It is saved **paused** and sends nothing until `set_automation_enabled` turns it on. Instagram lets a comment get one message before the person taps, so a post automation should have an opening message; the rest is sent after the tap. Attachments and card images use media library assets.

### `update_automation`

Changes an automation. Fields left out keep their saved value; fields sent replace it whole. To add or remove posts, read the trigger with `get_automation` and send it back with the target list changed. An automation that is on stays on, and the change applies everywhere it runs at once.

### `set_automation_enabled`

Turns an automation on or pauses it. Turning it on needs a connected Instagram account that can auto-post.

### `delete_automation`

Deletes an automation permanently. To stop it for now, pause it with `set_automation_enabled` instead. Replies already sent stay sent.
