# Security Policy

The Planoly MCP server is a hosted service operated by Planoly. This repository contains its public documentation and registry metadata.

## Reporting a vulnerability

Please **do not** open a public issue for security problems.

- Preferred: use GitHub's [private vulnerability reporting](https://github.com/Planoly/planoly-mcp/security/advisories/new) on this repository.
- Alternatively, contact us through the [Planoly help center](https://support.planoly.com) and ask for your report to be routed to the engineering security contact.

Include the endpoint or tool involved, reproduction steps, and impact. Never include live access tokens in a report.

## Scope

Reports about the hosted MCP service (`mcp.planoly.com`), its OAuth flow (`app.planoly.com/api/auth`), and the tool surface documented here are in scope. General Planoly product security reports are also welcome and will be routed internally.

## Design notes

- Tokens are user-scoped OAuth 2.1 bearer tokens; workspace membership is enforced on every workspace-scoped call.
- Tool output is an explicit allowlist of fields — provider tokens, refresh metadata, and internal identifiers are never emitted.
- Signed URLs appear only where contracted (asset upload, report share links) and are scoped and expiring.
- `create_asset_from_url` fetches only from an allowlist of known hosts; the signed-upload flow only receives bytes, leaving no server-side request surface.
