# MobilityMCP and TourismMCP

Two hosted remote MCP servers, run by PROJEKTIONISTEN. **This repository is
documentation only** — the servers themselves are operated by us; there is no
code to install and nothing to run.

<!-- servers:begin (generated) -->

| Server | What it answers | Endpoint |
|---|---|---|
| [MobilityMCP](mobilitymcp/README.md) | Public transport in Germany (ÖPNV, Fahrplan): journeys, departures, disruptions, nearby sharing. | `https://ai.projektionisten.eu/mmcp` |
| [TourismMCP](tourismmcp/README.md) | Travel in Germany (Tourismus): sights, events, opening hours and prices, weather and tides. | `https://ai.projektionisten.eu/tmcp` |

<!-- servers:end -->

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

## Directories

**MobilityMCP** · [Smithery](https://smithery.ai/servers/kaufmann/MobilityMCP) · [Glama](https://glama.ai/mcp/connectors/eu.projektionisten/mobility)

[![MobilityMCP MCP connector – tool definition quality and endpoint health on Glama](https://glama.ai/mcp/connectors/eu.projektionisten/mobility/badges/score.svg)](https://glama.ai/mcp/connectors/eu.projektionisten/mobility)

**TourismMCP** · [Glama](https://glama.ai/mcp/connectors/eu.projektionisten/tourism)

[![TourismMCP MCP connector – tool definition quality and endpoint health on Glama](https://glama.ai/mcp/connectors/eu.projektionisten/tourism/badges/score.svg)](https://glama.ai/mcp/connectors/eu.projektionisten/tourism)

## Support, legal

* Questions, problems, accounts: <support@projektionisten.de>
* Security reports: [SECURITY.md](SECURITY.md)

<!-- legal:begin (generated) -->

[Imprint](https://www.projektionisten.de/impressum) · [Privacy policy](https://www.projektionisten.de/mcp/mcp-datenschutz) (German) · [Terms of use](https://www.projektionisten.de/mcp/mcp-nutzungsbedingungen) (German)

<!-- legal:end -->
