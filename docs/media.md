# How media works

The model never carries media bytes through inference — an LLM can't faithfully relay binary, and inlined base64 would blow up the context window. Instead, posts reference media by `assetId`, and assets get created one of three ways depending on where the media lives and what the client can do:

## 1. Media already in Planoly

For anything the user has already uploaded, imported, or used in another post:

- `list_media_library` — browse/search the media library and take an `assetId` from an item's `assets[]`.
- `list_post_groups` — every post's media items include their `assetId`, so an existing post's media can be reused in a new post.

Works from **any** MCP client; the bytes entered Planoly through the app, not the agent.

## 2. Media at a URL

`create_asset_from_url` has the **server** fetch the file — only a URL crosses the wire, so this also works from any client. The source host must be on the supported allowlist:

- Instagram / Facebook CDNs (re-importing social media)
- Planoly's own hosts
- Supported AI-generation output hosts
- Design-tool export hosts (e.g. Canva)

Arbitrary URLs are rejected. If a generator only returns base64, steer it to a URL-returning mode or upload the bytes via option 3.

## 3. Local bytes via signed upload

For clients that can execute code / make HTTP requests (Claude Code, or claude.ai / Claude Desktop with a sandbox):

1. `request_asset_upload` → returns an `assetId` and a short-lived signed `uploadUrl`.
2. The client `PUT`s the raw bytes to `uploadUrl`.
3. `confirm_asset_upload` → server verifies the object and marks the asset ready.

Not available from a plain chat turn with no code execution.

## Seeing media

Tool results return CDN URLs (`url`, `thumbnailUrl`) as text. Clients that can't fetch URLs can call `get_media_preview` to receive an actual inline thumbnail image (one asset per call; videos return their cover frame).

## Notes

- Video metrics (duration, dimensions) are measured server-side during transcoding. A freshly uploaded video may schedule before all metrics exist; the platform API remains the final validation backstop.
- `create_post` backfills each media item's width/height/duration/fileSize from the asset, so posts created via MCP match posts created in the app.
