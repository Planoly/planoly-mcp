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
The tool maps to a Planoly premium feature (media library, analytics, TikTok trending sounds, product tagging, inbox). The error includes a link to upgrade. For the inbox, `list_workspaces` shows which workspaces include it (`inboxIncluded`).

**`POST_GROUP_CREATION_LIMIT_REACHED`.**
The workspace is on the free plan and has used its monthly post allowance. The error says how many are used and remaining, when the allowance resets, and links to upgrade. `get_account_billing` with the `workspaceId` shows the allowance.

**`RATE_LIMIT_REACHED`.**
Too many calls in a short time. Wait a moment and retry.

**`PROVIDER_ERROR` from `send_inbox_reply`.**
Instagram refused the reply. The message gives Instagram's reason and how many messages went out; the reply waits in the Planoly inbox to retry or discard. Read the conversation with `get_inbox_conversation` before sending again.

**A post saved but `willAutoPublish` is `false`.**
The channel can't auto-publish (it's reminder-only, or its connection needs fixing), so the user will get a reminder to post it manually. When reconnecting would fix it, the result includes a `connectUrl`. `list_channels` shows each channel's `canAutoPublish` and connection state.

**Why did my post fail?**
`list_post_groups` with `status: failed` returns each failed post's `latestError`.

**Validation error when scheduling.**
Auto-publish posts are checked against the target platform's content rules (media count/type, aspect ratio, video duration, caption length, …) at schedule time — the error lists every violation so it can be fixed and retried. Drafts and reminder-only posts are not content-gated.

**Caption rejected on create/update even for a draft.**
Caption length is enforced up front for every status, per platform, because an agent has no on-screen character counter. The error states the exact overage (e.g. "caption is 600 characters, 100 over the 500 limit").

**`create_asset_from_url` rejects my URL.**
Only HTTPS links to Instagram/Facebook media, Planoly media or its stock library, Canva exports, or Higgsfield/DALL·E output are accepted. Download the file and use the signed-upload flow instead, or add the media through the Planoly app.

**`USER_NOT_FOUND` from `get_instagram_account_posts`.**
The target handle is private, a personal (non-business/creator) account, or doesn't exist. Business Discovery only works for public Business/Creator accounts.

**`get_instagram_account_posts` returns a post with `mediaUrl: null`.**
Instagram withholds the file for some posts (e.g. reels with licensed music). That's an Instagram limit; the post's other details still come back.

**Instagram analytics tools return an error.**
They need a connected Instagram account that can serve insights: a Facebook-linked business account, or an Instagram Login account that granted the insights permission.

**TikTok, LinkedIn, or Pinterest analytics return an error.**
They need a channel for that network that auto-publishes — reminder-only channels have no analytics access. Personal LinkedIn profile analytics aren't available to every profile yet.

**An inbox DM won't send.**
Instagram only lets a DM through within its messaging window: 24 hours after the person's last message (7 days when the human-agent tag applies). `canReplyUntil` on the conversation shows when it closes; `null` means it's closed. DMs and private replies also need the account's messaging permission — `list_channels` flags accounts where Instagram is refusing messages and what to fix.

**An automation is on but sends nothing.**
Check the automation's `warnings`: an Instagram account that didn't grant a needed permission sends nothing until it's reconnected at the given `fixUrl`.

## Still stuck?

- Check the [Planoly help center](https://support.planoly.com).
- Open an issue on this repository with the client you're using, the tool called, and the error text (never include tokens).
