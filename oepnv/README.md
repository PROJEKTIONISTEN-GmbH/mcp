# PROJEKTIONISTEN ÖPNV

<!-- summary:begin (generated) -->

Getting there and what to do when you arrive, as one MCP tool set: journeys, departure boards and disruption reports for public transport in Germany, sights and curated travel records with opening hours, prices and events, weather and tides — and the place, address and time resolution the rest of them need.

<!-- summary:end -->

* **Endpoint:** `https://ai.projektionisten.eu/oepnv`
* **Transport:** MCP over streamable HTTP
* **Credential:** OAuth 2.1, or an API key as `Authorization: Bearer <your-key>`
* **Registry entry:** [`server.json`](server.json)

PROJEKTIONISTEN ÖPNV is the same data as [MobilityMCP](../mobilitymcp/README.md) and
[TourismMCP](../tourismmcp/README.md), cut for somebody who installs **one** app
in an assistant and then asks a question that does not respect the cut: *"Day
trip to Norderney — which train, what is the weather doing, when is the tide
out, and what is there to see?"* Every tool below is one of the two developer
servers' own, under the same name and the same description. It is a selection,
never a second dialect — and the account, the allowance and the key are the same
ones.

If you are building something and want the smaller, sharper tool set, take the
developer server for the half you need instead.

## What it reaches, and where it stops

This matters more than the feature list, because an assistant that does not know
where the data ends invents the rest.

* **Journeys are Germany-wide.** Timetable and real-time data cover the whole
  country.
* **Disruption reports cover Lower Saxony, Bremen and Hamburg.** Outside that
  the tool answers empty and says so in the answer itself. An empty answer there
  is **not** an all-clear.
* **Travel content is Germany-wide in breadth, not in depth.** The
  OpenStreetMap layer — places, addresses, sights — covers the country. The
  curated layer with editorial descriptions, opening hours, prices and events is
  the **Niedersachsen-Hub**, and it is densest in Lower Saxony. A region outside
  it answers thinner rather than differently.
* **Tides exist on the coast.** Inland there is no gauge and no tide.
* **It books nothing and buys nothing.** Every tool reads a source and changes
  nothing: there are no tickets, no reservations, and nothing to cancel.

## The tools

<!-- tools:begin (generated from the server's tools/list) -->

20 tools. The full description of each — arguments, limits, what may be quoted from an answer — is in [`tools.md`](tools.md).

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
| Ort im Detail | `get_poi_details` |
| Sehenswürdigkeiten im Gebiet | `expand_kg_pois` |
| Kuratierte Tourismus-Suche | `search_tourism` |
| Kuratierter Eintrag im Detail | `get_tourism_details` |
| Aktuelles Wetter | `get_current_weather` |
| Wettervorhersage | `get_weather_forecast` |
| Gezeiten abfragen | `get_tide` |
| Zeitangabe auflösen | `link_datetime` |
| Verbindung anzeigen | `show_connections` |
| Abfahrtstafel anzeigen | `show_departures` |
| Verbindung schlank anzeigen | `show_connections_slim` |
| Anleitung lesen | `get_usage_guide` |

<!-- tools:end -->

The server also publishes its usage guide and its source list as MCP resources,
so an assistant can read both without being told.

## Sources and licences

The answers carry third-party data. The licences require the source to be named
where the data is shown — also when a model sits in between. Each answer that
carries sourced data repeats these notices machine-readably in its
`attribution` field, and each curated record carries its own `license` block
naming the licence **and its holder**, who is the author of that record and is
named beside the source, never instead of it.

| Source | Licence | Notice to pass on |
|---|---|---|
| Timetable and real-time data of the German transport associations | per feed (<https://www.opendata-oepnv.de/>) | each answer's own `attribution` |
| OpenStreetMap | ODbL-1.0 (<https://www.openstreetmap.org/copyright>) | `© OpenStreetMap contributors` |
| Curated travel content — Niedersachsen-Hub, Deutsche Zentrale für Tourismus | per record (<https://www.germany.travel/>) | each record's own, in its `license` block |
| Deutscher Wetterdienst | GeoNutzV (<https://www.dwd.de/DE/service/copyright/copyright_node.html>) | `Quelle: Deutscher Wetterdienst` |
| PEGELONLINE — Wasserstraßen- und Schifffahrtsverwaltung des Bundes | DL-DE/Zero-2.0 (<https://www.pegelonline.wsv.de/gast/nutzungsbedingungen>) | nothing to name |

## Support

<support@projektionisten.de>

<!-- legal:begin (generated) -->

[Imprint](https://www.projektionisten.de/impressum) · [Privacy policy](https://www.projektionisten.de/mcp/mcp-datenschutz) (German) · [Terms of use](https://www.projektionisten.de/mcp/mcp-nutzungsbedingungen) (German)

<!-- legal:end -->
