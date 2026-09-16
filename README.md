# MobilityMCP and TourismMCP

Two hosted remote MCP servers, run by PROJEKTIONISTEN. **This repository is
documentation only** — the servers themselves are operated by us; there is no
code to install and nothing to run.

| Server | What it answers | Endpoint |
|---|---|---|
| [MobilityMCP](mobilitymcp/README.md) | public transport in Germany: journeys, departure boards, line courses, disruption reports | `https://ai.projektionisten.eu/mmcp` |
| [TourismMCP](tourismmcp/README.md) | sights and places, curated tourism records, weather, tides, sharing availability | `https://ai.projektionisten.eu/tmcp` |

Both speak MCP over **streamable HTTP**. Both are metered per account.

## They are deliberately not one server

MobilityMCP answers timetable questions and nothing else — no weather, no
tides, no tourism content. TourismMCP answers trip questions and
holds no timetable service: journeys, departure boards and disruption reports
are not available there. If your agent needs both, connect both; they share one
account and the places they resolve are resolved by the same tools, so an id
from one is an id the other understands.

Each server states its own reach in its handshake, and each tool states its own
in its description. Where the data stops, the answer says so rather than filling
the gap — see the two READMEs.

## Connecting

Ready-made configuration for the common clients is in [`clients/`](clients/):

* [Claude Desktop](clients/claude-desktop.json) — the shortest path there is
  *Settings → Connectors → Add custom connector* and the endpoint URL; the
  JSON is for a build without that dialog.
* [Claude Code](clients/claude-code.md)
* [VS Code](clients/vscode.json)
* [Cursor](clients/cursor.md) — including one-click install links
* [Antigravity](clients/antigravity.md)

Anything else that speaks MCP over streamable HTTP works too: point it at the
endpoint above. Tool-running platforms such as n8n take the same two values —
the endpoint URL and the credential — in whatever their MCP client node calls
them.

### Credentials

Two ways in, and a client picks whichever it supports:

* **OAuth 2.1.** The endpoints publish the login signpost a client needs
  (`/.well-known/oauth-protected-resource/…`), so a client that supports remote
  MCP servers with OAuth — Claude and Claude Code among them — finds the way to
  sign in by itself. Nothing to paste.
* **An API key**, sent as `Authorization: Bearer <your-key>`, for a client that
  takes a header.

`initialize` and `tools/list` are answered without a credential, so a client may
read what a server offers before anybody signs in. Everything that returns data
needs one.

**Getting an account:** see *Zugang* on
<https://ai.projektionisten.eu/mcp-landingpage/#zugang>, or write to
<support@projektionisten.de>.

## Where the data comes from

Open data of third parties, under licences that require the source to be named
where the data is shown — also when a model sits in between. Each server's
README lists its sources and their licences, each answer carries the same notice
machine-readably in an `attribution` field, and every server publishes the full
list as an MCP resource (`attribution://…/sources`). **If you show data from
these servers, pass the notice on.**

## Support, legal

* Questions, problems, accounts: <support@projektionisten.de>
* Security reports: [SECURITY.md](SECURITY.md)
* Imprint: <https://www.projektionisten.de/impressum>
* Privacy policy: <https://www.projektionisten.de/datenschutz>

Terms for the MCP offering are being drawn up; until they are published, the
terms in place are the ones agreed with your account.
