# Antigravity

Antigravity keeps its MCP servers in `mcp_config.json` — globally under
`~/.gemini/config/`, or per workspace under `.agents/`. In the IDE: the **…**
menu at the top of the agent panel → *MCP Servers* → *Manage MCP Servers* →
*View raw config*.

Note the field name: Antigravity reads **`serverUrl`** for a remote server,
where Cursor and Claude Code read `url`.

```json
{
  "mcpServers": {
    "mobilitymcp": {
      "serverUrl": "https://ai.projektionisten.eu/mmcp"
    },
    "tourismmcp": {
      "serverUrl": "https://ai.projektionisten.eu/tmcp"
    }
  }
}
```

With an API key instead of the OAuth login, add the header:

```json
"headers": { "Authorization": "Bearer <your-key>" }
```

## Checking

Reopen the MCP panel after saving. MobilityMCP lists 13 tools, TourismMCP 16. If
a server shows none, it has not signed in yet — the tool list is public, the
answers are not.
