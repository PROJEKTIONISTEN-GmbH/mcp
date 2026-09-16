# Claude Code

## Adding the servers

```bash
claude mcp add --transport http mobilitymcp https://ai.projektionisten.eu/mmcp
claude mcp add --transport http tourismmcp  https://ai.projektionisten.eu/tmcp
```

With an API key instead of the OAuth login:

```bash
claude mcp add --transport http mobilitymcp https://ai.projektionisten.eu/mmcp \
  --header "Authorization: Bearer <your-key>"
```

## Signing in

```
/mcp
```

Pick the server and follow the browser login. Claude Code keeps the token and
refreshes it; `claude mcp login <name>` does the same from the shell, and
`claude mcp logout <name>` forgets it again.

## As a project file

Checked in beside the project, so everybody working on it gets the same two
servers — `.mcp.json`:

```json
{
  "mcpServers": {
    "mobilitymcp": {
      "type": "http",
      "url": "https://ai.projektionisten.eu/mmcp"
    },
    "tourismmcp": {
      "type": "http",
      "url": "https://ai.projektionisten.eu/tmcp"
    }
  }
}
```

Credentials do not belong in that file — it is shared. Leave the login to OAuth,
or keep the key in your own user-level configuration.

## Checking

```
/mcp
```

MobilityMCP lists 13 tools, TourismMCP 16. The list is answered without a
credential, so it appears before you have signed in; the answers do not.
