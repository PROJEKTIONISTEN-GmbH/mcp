# TourismMCP

[![TourismMCP MCP connector – tool definition quality and endpoint health on Glama](https://glama.ai/mcp/connectors/eu.projektionisten/tourism/badges/score.svg)](https://glama.ai/mcp/connectors/eu.projektionisten/tourism)

<!-- summary:begin (generated) -->

Travel and day-trip knowledge as one MCP tool set: sights and place details, curated tourism records with opening hours, prices and events, weather, tides, and a vicinity search that answers with nearby stops, places and sharing offers — the last of those with the free stock currently reported for them.

<!-- summary:end -->

* **Endpoint:** `https://ai.projektionisten.eu/tmcp`
* **Transport:** MCP over streamable HTTP
* **Credential:** OAuth 2.1, or an API key as `Authorization: Bearer <your-key>`
* **Registry entry:** [`server.json`](server.json)
* **Setup:** [`llms-install.md`](llms-install.md) · **Prompts:** [`examples.md`](examples.md)
* **Listed in:** [Glama](https://glama.ai/mcp/connectors/eu.projektionisten/tourism)

## What it reaches, and where it stops

This matters more than the feature list, because an agent that does not know
where the data ends invents the rest.

* **Breadth is Germany-wide, depth is not.** The OpenStreetMap layer — places,
  addresses, sights and their tags — covers the whole country. The curated
  layer, the one with editorial descriptions, prices, opening hours, events and
  tours, is the **Niedersachsen-Hub**, and it is densest in Lower Saxony. A
  region outside it answers thinner rather than differently.
* **Tides exist on the coast.** `get_tide` reads gauges; inland there is no
  gauge and no tide.
* **No timetable service.** Journeys, departure boards and disruption reports
  are not available here; that is
  [MobilityMCP](../mobilitymcp/README.md). Sharing stock is the one mobility
  fact this server does carry, and it comes through `nearby`.
* **No writing.** Every tool queries a source and changes nothing.

Four things the server insists on, and that a good prompt does not fight:

1. **Resolve first, then ask.** The content and environment tools take
   coordinates or ids. `search_place` turns an area into coordinates,
   `resolve_location` turns a name into an id, `link_datetime` turns „morgen"
   into an instant. A guessed coordinate lands in a different part of the
   country.
2. **A value belongs to the place it was asked for.** A tide time from the
   nearest gauge is that gauge's time — name the gauge rather than attributing
   its time to the town somebody asked about.
3. **Pass the source on.** Where an answer carries `attribution` or `license`,
   naming the source is a condition of the data, not a courtesy. The `license`
   block beside a curated record names the licence **and its holder**; the
   holder is the author of that record and is named beside the source, never
   instead of it.
4. **Say nothing a record does not carry.** A suitability — "good for children",
   "barrier-free" — is a field or it is not said.

## The tools

<!-- tools:begin (generated from the server's tools/list) -->

14 tools. The full description of each — arguments, limits, what may be quoted from an answer — is in [`tools.md`](tools.md).

| Tool | Name |
|---|---|
| Stadt oder Region suchen | `search_place` |
| Ort auflösen | `resolve_location` |
| Umgebung einer Koordinate | `nearby` |
| Koordinate benennen | `reverse_geocode` |
| Ort im Detail | `get_poi_details` |
| Sehenswürdigkeiten im Gebiet | `expand_kg_pois` |
| Kuratierte Tourismus-Suche | `search_tourism` |
| Kuratierter Eintrag im Detail | `get_tourism_details` |
| Aktuelles Wetter | `get_current_weather` |
| Wettervorhersage | `get_weather_forecast` |
| Gezeiten abfragen | `get_tide` |
| Zeitangabe auflösen | `link_datetime` |
| Nennung zuordnen | `link_dynamic` |
| Anleitung lesen | `get_usage_guide` |

<!-- tools:end -->

Beyond the tools, the server publishes reference material as MCP resources: the
usage guide (`guide://tourismmcp/usage`), the answer schema of every tool
(`schema://tmcp/<tool>`) for a client that validates rather than reads, and the
source list below (`attribution://tourismmcp/sources`).

## Sources and licences

The answers carry third-party data. The licences require the source to be named
where the data is shown — also when a model sits in between. Each answer that
carries sourced data repeats these notices machine-readably in its
`attribution` field, and each curated record carries its own `license` block.

| Source | Licence | Notice to pass on |
|---|---|---|
| OpenStreetMap | ODbL-1.0 (<https://www.openstreetmap.org/copyright>) | `© OpenStreetMap contributors` |
| Curated tourism content — Niedersachsen-Hub, Deutsche Zentrale für Tourismus | per record (<https://www.germany.travel/>) | each record's own, in its `license` block |
| Deutscher Wetterdienst | GeoNutzV (<https://www.dwd.de/DE/service/copyright/copyright_node.html>) | `Quelle: Deutscher Wetterdienst` |
| PEGELONLINE — Wasserstraßen- und Schifffahrtsverwaltung des Bundes | DL-DE/Zero-2.0 (<https://www.pegelonline.wsv.de/gast/nutzungsbedingungen>) | nothing to name |

The curated row says *per record* for a reason: that collection bundles records
under differing terms, and each hit carries its own notice — licence, rights
holder, and the wording to repeat. One blanket notice would be wrong for part of
it.

## Support

<support@projektionisten.de>

<!-- legal:begin (generated) -->

[Imprint](https://www.projektionisten.de/impressum) · [Privacy policy](https://www.projektionisten.de/mcp/mcp-datenschutz) (German) · [Terms of use](https://www.projektionisten.de/mcp/mcp-nutzungsbedingungen) (German)

<!-- legal:end -->
