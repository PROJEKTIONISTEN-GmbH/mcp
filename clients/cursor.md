# Cursor

## One click

[**Add MobilityMCP to Cursor**](cursor://anysphere.cursor-deeplink/mcp/install?name=mobilitymcp&config=eyJ1cmwiOiJodHRwczovL2FpLnByb2pla3Rpb25pc3Rlbi5ldS9tbWNwIn0=)

[**Add TourismMCP to Cursor**](cursor://anysphere.cursor-deeplink/mcp/install?name=tourismmcp&config=eyJ1cmwiOiJodHRwczovL2FpLnByb2pla3Rpb25pc3Rlbi5ldS90bWNwIn0=)

Cursor opens with the server pre-filled; confirm, and sign in when it asks.

The two links carry nothing but the endpoint — the `config` parameter is the
base64 of the two-line configuration below, which you can decode and read
before clicking if you would rather see it than trust it.

## By hand

Add the entries to Cursor's MCP configuration (Settings → MCP → *Add new MCP
server*, or the raw `mcp.json` behind it):

```json
{
  "mcpServers": {
    "mobilitymcp": { "url": "https://ai.projektionisten.eu/mmcp" },
    "tourismmcp": { "url": "https://ai.projektionisten.eu/tmcp" }
  }
}
```

With an API key instead of the OAuth login, add the header:

```json
"headers": { "Authorization": "Bearer <your-key>" }
```

## Checking

Open the MCP panel. MobilityMCP lists 13 tools, TourismMCP 16. If a server shows
none, it has not signed in yet — the tool list is public, the answers are not.
