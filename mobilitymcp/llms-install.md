# Installing MobilityMCP — for an agent doing the setup

One remote MCP server. There is nothing to download, nothing to build and no
process to run: point a client at the endpoint and let it sign in.

```
name:      mobilitymcp
endpoint:  https://ai.projektionisten.eu/mmcp
transport: streamable-http
auth:      OAuth 2.1 (preferred) — or an API key as "Authorization: Bearer <key>"
```

## Step 1 — write the client configuration

**A client that reads `mcpServers` with a `type`** (Claude Code, and anything
following the same shape):

```json
{
  "mcpServers": {
    "mobilitymcp": {
      "type": "http",
      "url": "https://ai.projektionisten.eu/mmcp"
    }
  }
}
```

**VS Code** uses `servers` instead of `mcpServers`:

```json
{
  "servers": {
    "mobilitymcp": {
      "type": "http",
      "url": "https://ai.projektionisten.eu/mmcp"
    }
  }
}
```

**Antigravity** uses `serverUrl` instead of `url`:

```json
{
  "mcpServers": {
    "mobilitymcp": {
      "serverUrl": "https://ai.projektionisten.eu/mmcp"
    }
  }
}
```

With an API key instead of OAuth, add the header:

```json
"headers": { "Authorization": "Bearer <your-key>" }
```

Ready-made files for the common clients are in [`../clients/`](../clients/).

## Step 2 — sign in

If the client supports OAuth for remote MCP servers, it finds the login by
itself: the endpoint publishes the signpost the protocol asks for, and the
client opens a browser once. In Claude Code that is `/mcp`, then pick the
server.

With an API key, there is nothing to sign in to — the header is the credential.

## Step 3 — check that it answers

Ask the client to list the tools. You should see **10**, beginning with
`search_place` and ending with `get_usage_guide`; the full list is in
[`tools.md`](tools.md). `tools/list` is answered without a credential, so this
step works even before the login does.

Then ask one real question, for instance:

> Wie komme ich morgen früh um 8 von Hannover Hauptbahnhof nach Braunschweig?

A good answer names departure and arrival times, the lines, and where to change
— all of it out of the tool result.

## If it does not work

* **The client tries to run the URL as a command.** The `type` field is
  missing. A `url` without `"type": "http"` is read as a local server in most
  clients.
* **Everything is refused with `401`.** The credential is missing or expired.
  With OAuth, sign in again; with a key, check the header spelling — it is
  `Authorization: Bearer <key>`.
* **The tool list is there but every call is refused.** The account has run out
  of its allowance, or it is not admitted at this endpoint. Ask
  <support@projektionisten.de>.
* **An answer is empty.** That is an answer, not a failure — read the tool's
  description for what commonly causes one, and pass the emptiness on honestly
  rather than filling it in.
