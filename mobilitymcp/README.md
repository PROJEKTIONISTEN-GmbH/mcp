# MobilityMCP

<!-- summary:begin (generated) -->

Public transport in Germany as one MCP tool set: journeys, departure boards, line courses and disruption reports, the place, address and time resolution they need, and a vicinity search that answers with nearby stops, places and sharing offers — the last of those with the free stock currently reported for them.

<!-- summary:end -->

* **Endpoint:** `https://ai.projektionisten.eu/mmcp`
* **Transport:** MCP over streamable HTTP
* **Credential:** OAuth 2.1, or an API key as `Authorization: Bearer <your-key>`
* **Registry entry:** [`server.json`](server.json)
* **Setup:** [`llms-install.md`](llms-install.md) · **Prompts:** [`examples.md`](examples.md)

## What it reaches, and where it stops

This matters more than the feature list, because an agent that does not know
where the data ends invents the rest.

* **Journeys, departures and line courses are Germany-wide.** They are planned
  on the nationwide timetable data set, long-distance services included.
* **Disruption reports cover Lower Saxony, Bremen and Hamburg.** Outside those,
  there is no data behind this tool at all. An empty disruption answer outside
  that area **is not an all-clear** — every answer says in its own `coverage`
  field what it can speak for, and your agent should pass that on rather than
  read silence as "no disruptions".
* **Nothing but mobility.** Weather, tides and tourism content are not here;
  that is [TourismMCP](../tourismmcp/README.md).
* **No writing.** Every tool queries a timetable and changes nothing.

Three things the server insists on, and that a good prompt does not fight:

1. **Resolve first, then ask.** `connections`, `departures` and `disruptions`
   take ids and coordinates, not free text. Turn a name into an id with
   `resolve_location`, an area into coordinates with `search_place`, and a
   phrase like „morgen früh" into an instant with `link_datetime`.
2. **An empty result is an intermediate state.** Each tool's description lists
   what commonly causes one — a filter that is too narrow, a time nothing runs
   at, an id the routing does not know.
3. **Quote only what an answer carries.** Lines, times, stops, platforms and
   disruption texts come out of a tool result, never out of the model.

## The tools

<!-- tools:begin (generated from the server's tools/list) -->

10 tools. The full description of each — arguments, limits, what may be quoted from an answer — is in [`tools.md`](tools.md).

| Tool | Name |
|---|---|
| Stadt oder Region suchen | `search_place` |
| Ort auflösen | `resolve_location` |
| Umgebung einer Koordinate | `nearby` |
| Koordinate benennen | `reverse_geocode` |
| Verbindung suchen | `connections` |
| Abfahrtstafel lesen | `departures` |
| Störungen abfragen | `disruptions` |
| Linienverlauf abfragen | `line_course` |
| Zeitangabe auflösen | `link_datetime` |
| Anleitung lesen | `get_usage_guide` |

<!-- tools:end -->

Three of them — the ones whose name begins with `show_` — answer the same data
*and* an application view beside it, for a host that draws rather than prints.
A host that does not draw ignores the view and reads the data.

Beyond the tools, the server publishes reference material as MCP resources: the
usage guide (`guide://mobilitymcp/usage`), the answer schema of every tool
(`schema://mmcp/<tool>`) for a client that validates rather than reads, and the
source list below (`attribution://mobilitymcp/sources`).

## Sources and licences

The answers carry third-party data. The licences require the source to be named
where the data is shown — also when a model sits in between. Each answer that
carries sourced data repeats these notices machine-readably in its
`attribution` field, and a rendered map image carries them in its pixels.

| Source | Licence | Notice to pass on |
|---|---|---|
| OpenStreetMap | ODbL-1.0 (<https://www.openstreetmap.org/copyright>) | `© OpenStreetMap contributors` |
| DELFI e.V. — nationwide timetable and real-time data | CC-BY-4.0 (<https://www.opendata-oepnv.de/>) | `CC BY 4.0 © DELFI e.V.` |
| MobiData BW (Nahverkehrsgesellschaft Baden-Württemberg) | DL-DE/BY-2.0 (<https://www.mobidata-bw.de/>) | `Datenquelle: MobiData BW, Nahverkehrsgesellschaft Baden-Württemberg (NVBW)` |
| Mobilitätsdatenmarktplatz go.Rheinland | per network (<https://mdd.gorheinland.com/>) | `Datenquelle: Mobilitätsdatenmarktplatz go.Rheinland` |
| The operators of the GBFS feeds behind the sharing stock | per network (<https://gbfs.org/>) | each network's own — every station in an answer carries it |
| stadtmobil carsharing | no open licence stated (<https://www.stadtmobil.de/>) | `Stationsdaten: stadtmobil` |
| Protomaps — the vector base-map build our tiles are cut from | BSD-3-Clause (<https://build.protomaps.com/>) | nothing to name |
| Natural Earth — the small-scale basis of the base map | public domain (<https://www.naturalearthdata.com/>) | nothing to name |

The two rows that say *per network* are the reason the notice travels in the
answer rather than in this table: those sources bundle data under differing
terms, and one blanket notice would be wrong for part of it.

## Support

<support@projektionisten.de>

<!-- legal:begin (generated) -->

[Imprint](https://www.projektionisten.de/impressum) · [Privacy policy](https://www.projektionisten.de/mcp/mcp-datenschutz) (German) · [Terms of use](https://www.projektionisten.de/mcp/mcp-nutzungsbedingungen) (German)

<!-- legal:end -->
