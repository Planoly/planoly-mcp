# Troubleshooting

## Connection & auth

**The browser sign-in window never opens / the client hangs on connect.**
Make sure the client supports remote MCP servers with OAuth (streamable HTTP transport). For stdio-only clients, bridge through `mcp-remote` — see the README's "Other clients" section.

**Auth loop or stale credentials with `mcp-remote`.**
Clear the bridge's local auth cache and restart the client:

```bash
rm -rf ~/.mcp-auth
```

**`401 Unauthorized` on every call after connecting previously.**
The grant was revoked or expired. Disconnect and reconnect the server in your client's settings to re-run the OAuth flow.

**Which account am I acting as?**
The Planoly account you signed in with during the OAuth flow. Tokens are user-scoped; `list_workspaces` shows exactly what that user can touch.

## Tool errors

**`FORBIDDEN` on a workspace tool.**
The signed-in user isn't a member of that `workspaceId`. Call `list_workspaces` and use an id from the result.

**`SUBSCRIPTION_REQUIRED`.**
The tool maps to a Planoly premium feature (media library, analytics, TikTok trending sounds, product tagging). The error includes a billing link to upgrade the workspace.

**Validation error when scheduling.**
Auto-publish posts are checked against the target platform's content rules (media count/type, aspect ratio, video duration, caption length, …) at schedule time — the error lists every violation so it can be fixed and retried. Drafts and reminder-only posts are not content-gated.

**Caption rejected on create/update even for a draft.**
Caption length is enforced up front for every status, per platform, because an agent has no on-screen character counter. The error states the exact overage (e.g. "caption is 600 characters, 100 over the 500 limit").

**`create_asset_from_url` rejects my URL.**
The source host isn't on the import allowlist (Instagram/Facebook CDNs, Planoly, supported AI-generation hosts, design-tool exports like Canva). Download the file and use the signed-upload flow instead, or add the media through the Planoly app.

**`USER_NOT_FOUND` from `get_instagram_account_posts`.**
The target handle is private, a personal (non-business/creator) account, or doesn't exist. Business Discovery only works for public Business/Creator accounts.

**Instagram analytics tools return an error.**
They require a connected Instagram **Business** account in the workspace. Personal/creator-basic connections don't expose insights.

## Still stuck?

- Check the [Planoly help center](https://support.planoly.com).
- Open an issue on this repository with the client you're using, the tool called, and the error text (never include tokens).
