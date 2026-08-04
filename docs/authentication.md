# Authentication

The Planoly MCP server uses the standard MCP authorization flow. Any spec-compliant MCP client discovers and completes it automatically — you sign in with your existing Planoly account in a browser window and approve access.

## How it works

1. The client calls `https://mcp.planoly.com/mcp` and receives a `401` with a `WWW-Authenticate` challenge pointing at the protected resource metadata.
2. Protected resource metadata at `https://mcp.planoly.com/.well-known/oauth-protected-resource/mcp` names the authorization server: `https://app.planoly.com/api/auth`.
3. The client runs an OAuth 2.1 **authorization code** flow (with PKCE and dynamic client registration) against that server — this is the browser window where you log in to Planoly and consent.
4. The client receives an access token (and a refresh token when `offline_access` is granted) and retries the MCP request as a `Bearer` token.

Supported grant types: `authorization_code`, `refresh_token`.
Supported scopes (identity only): `openid`, `profile`, `email`, `offline_access`.

## Token model

- Tokens are scoped to your **Planoly user**, not to a workspace.
- Every workspace-scoped tool takes an explicit `workspaceId` and enforces workspace membership per call — a token can only act on workspaces its user belongs to, with that user's role.
- Access tokens expire; clients refresh silently with the refresh token. Revoking access (or signing out) invalidates the grant.

## Resetting auth state

Most clients re-run the browser flow automatically when tokens expire or are revoked. If a client gets stuck:

- **Claude Code / claude.ai / Claude Desktop connectors**: disconnect and reconnect the server from settings.
- **`mcp-remote` bridge**: clear the local cache and restart the client:

  ```bash
  rm -rf ~/.mcp-auth
  ```
