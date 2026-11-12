# MobilityMCP

[![MobilityMCP MCP connector – tool definition quality and endpoint health on Glama](https://glama.ai/mcp/connectors/eu.projektionisten/mobility/badges/score.svg)](https://glama.ai/mcp/connectors/eu.projektionisten/mobility)

<!-- summary:begin (generated) -->

Public transport in Germany, Austria and Switzerland as one MCP tool set: journeys, departure boards and line courses, disruption reports, the place, address and time resolution they need, and a vicinity search that answers with nearby stops, places and sharing offers — the last of those with the free stock currently reported for them.

<!-- summary:end -->

* **Endpoint:** `https://ai.projektionisten.eu/mmcp`
* **Transport:** MCP over streamable HTTP
* **Credential:** OAuth 2.1, or an API key as `Authorization: Bearer <your-key>`
* **Registry entry:** [`server.json`](server.json)
* **Setup:** [`llms-install.md`](llms-install.md) · **Prompts:** [`examples.md`](examples.md)
* **Listed in:** [Smithery](https://smithery.ai/servers/kaufmann/MobilityMCP) · [Glama](https://glama.ai/mcp/connectors/eu.projektionisten/mobility)

## What it reaches, and where it stops

This matters more than the feature list, because an agent that does not know
where the data ends invents the rest.

* **Journeys, departures and line courses cover Germany, Austria, Switzerland
  and South Tyrol.** They are planned on the timetables of all three countries
  and of South Tyrol (the Italian province), long-distance and cross-border
  services included — München to Bozen over the Brenner, for one. In Germany and
  Switzerland, and on South Tyrol's buses, they come with real-time data —
  delays, cancellations, platform changes, notices. In Austria and on South
  Tyrol's trains they come from the timetable alone — **no real time**. Every
  ride names its source, and each answer says when each source's timetable was
  last updated. A place outside these regions — the rest of Italy included — is
  not covered.
* **Beyond the German border, `resolve_location` finds the places.** It resolves
  stops, addresses and places in Austria, Switzerland and South Tyrol; `search_place`,
  `reverse_geocode` and `nearby` search Germany only, and the sharing stock and
  park-and-ride sites come from German feeds.
* **Disruption reports cover Lower Saxony, Bremen, Hamburg and Switzerland** —
  Switzerland while its sources are current; elsewhere, Austria included, there
  is no data behind this tool. An empty disruption answer outside those regions
  **is not an all-clear** — every answer says in its own `coverage` field what
  it can speak for, and your agent should pass that on rather than read silence
  as "no disruptions".
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

13 tools. The full description of each — arguments, limits, what may be quoted from an answer — is in [`tools.md`](tools.md).

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
| Verbindung anzeigen | `show_connections` |
| Abfahrtstafel anzeigen | `show_departures` |
| Verbindung schlank anzeigen | `show_connections_slim` |

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
| BKG (Bundesamt für Kartographie und Geodäsie), Verwaltungsgebiete 1:25 000 (VG25) — the official areas of Germany a place is named by | CC-BY-4.0 (<https://creativecommons.org/licenses/by/4.0/>) | `© BKG (2026) CC BY 4.0, Datenquellen: https://sgx.geodatenzentrum.de/web_public/gdz/datenquellen/datenquellen_vg25.pdf (Daten verändert)` |
| BKG (Bundesamt für Kartographie und Geodäsie), Verwaltungsgebiete 1:250 000 mit Einwohnerzahlen (VG250-EW) — the population of those areas | DL-DE/BY-2.0 (<https://www.govdata.de/dl-de/by-2-0>) | `© BKG (2026) dl-de/by-2-0, Datenquellen: https://sgx.geodatenzentrum.de/web_public/gdz/datenquellen/datenquellen_vg_nuts.pdf (Daten verändert)` |
| DELFI e.V. — nationwide timetable and real-time data | CC-BY-4.0 (<https://www.opendata-oepnv.de/>) | `CC BY 4.0 © DELFI e.V.` |
| Mobilitätsverbünde Österreich OG — the timetables of the Austrian transport associations | Datenlizenz Mobilitätsverbünde Österreich (<https://data.mobilitaetsverbuende.at/de/data-sets>) | `Datenquelle: Mobilitätsverbünde Österreich OG` |
| opentransportdata.swiss — the Swiss timetable | Nutzungsbedingungen opentransportdata.swiss (<https://opentransportdata.swiss/de/terms-of-use/>) | `Datenquelle: opentransportdata.swiss` |
| STA – Südtiroler Transportstrukturen AG — the timetable of South Tyrol, published by the Open Data Hub | CC0-1.0 (<https://gtfs.api.opendatahub.com/v1/dataset/sta-time-tables>) | `Datenquelle: STA – Südtiroler Transportstrukturen AG (Open Data Hub)` |
| SkyAlps — the flights from Bolzano, published by the Open Data Hub | CC0-1.0 (<https://gtfs.api.opendatahub.com/v1/dataset/skyalps-flight-data>) | `Datenquelle: SkyAlps (Open Data Hub)` |
| ummadum — the shared rides offered in South Tyrol, via Amarillo, published by the Open Data Hub | CC0-1.0 (<https://gtfs.api.opendatahub.com/v1/dataset/ummadumm-data>) | `Datenquelle: ummadum (Amarillo, Open Data Hub)` |
| Administration des transports publics (Luxembourg) — the timetable of Luxembourg, published on data.public.lu | CC0-1.0 (<https://data.public.lu/en/datasets/horaires-et-arrets-des-transport-publics-netex/>) | `Datenquelle: Administration des transports publics Luxembourg (data.public.lu)` |
| Rejseplanen A/S — the timetable of Denmark, from Denmark's national access point | CC-BY-4.0 (<https://creativecommons.org/licenses/by/4.0/>) | `Datenquelle: Rejseplanen A/S (Dataudveksleren), CC BY 4.0` |
| Entur — the timetable of Norway with the national stop register | NLOD-2.0 (<https://data.norge.no/nlod/en/2.0>) | `Datenquelle: Entur (Norwegen), NLOD 2.0` |
| PKP Polskie Linie Kolejowe S.A. — timetable, operation and disruptions of the railway in Poland | terms of the Open Railway Data API (<https://pdp-api.plk-sa.pl/api/v1/terms/html>) | `Datenquelle: PKP Polskie Linie Kolejowe S.A.` |
| NMBS/SNCB — the railway in Belgium, via the open-data portal of the Belgian Mobility Company | CC-BY-4.0 (<https://creativecommons.org/licenses/by/4.0/>) | `Datenquelle: NMBS/SNCB – Open Data (Belgian Mobility Company), CC BY 4.0` with the date of the version (see below) |
| STIB-MIVB — metro, tram and bus in Brussels, via the open-data portal of the Belgian Mobility Company | CC-BY-4.0 (<https://creativecommons.org/licenses/by/4.0/>) | `Datenquelle: STIB-MIVB – Open Data (Belgian Mobility Company), CC BY 4.0` with the date of the version (see below) |
| De Lijn — bus and tram in Flanders, via the open-data portal of the Belgian Mobility Company | CC-BY-4.0 (<https://creativecommons.org/licenses/by/4.0/>) | `Datenquelle: De Lijn – Open Data (Belgian Mobility Company), CC BY 4.0` with the date of the version (see below) |
| LETEC — bus in Wallonia, via the open-data portal of the Belgian Mobility Company | CC-BY-4.0 (<https://creativecommons.org/licenses/by/4.0/>) | `Datenquelle: LETEC – Open Data (Belgian Mobility Company), CC BY 4.0` with the date of the version (see below) |
| Samtrafiken — the timetable and the stop register of Sweden, via Trafiklab | CC0-1.0 (<https://creativecommons.org/publicdomain/zero/1.0/>) | `Datenquelle: Trafiklab/Samtrafiken` |
| OVapi — GTFS-NL, the timetable of the Netherlands | CC0-1.0 (<https://creativecommons.org/publicdomain/zero/1.0/>) | `Datenquelle: OVapi (GTFS-NL)` |
| CIS JŘ (Ministry of Transport of the Czech Republic) — the timetables of Czechia | CC0-1.0 (<https://creativecommons.org/publicdomain/zero/1.0/>) | `Datenquelle: Ministerstvo dopravy ČR – CIS JŘ` |
| Fluo Grand Est — the timetable of the Grand Est region: TER trains and buses, the interurban buses of the départements and the urban networks | Licence Ouverte 2.0 (<https://www.etalab.gouv.fr/licence-ouverte-open-licence/>) | `Datenquelle: Région Grand Est – Fluo Grand Est, Licence Ouverte 2.0` with the date of the version (see below) |
| MobiData BW (Nahverkehrsgesellschaft Baden-Württemberg) | DL-DE/BY-2.0 (<https://www.mobidata-bw.de/>) | `Datenquelle: MobiData BW, Nahverkehrsgesellschaft Baden-Württemberg (NVBW)` |
| Mobilitätsdatenmarktplatz go.Rheinland | per network (<https://mdd.gorheinland.com/>) | `Datenquelle: Mobilitätsdatenmarktplatz go.Rheinland` |
| The operators of the GBFS feeds behind the sharing stock | per network (<https://gbfs.org/>) | each network's own — every station in an answer carries it |
| Protomaps — the vector base-map build our tiles are cut from | BSD-3-Clause (<https://build.protomaps.com/>) | nothing to name |
| Natural Earth — the small-scale basis of the base map | public domain (<https://www.naturalearthdata.com/>) | nothing to name |

The two rows that say *per network* are the reason the notice travels in the
answer rather than in this table: those sources bundle data under differing
terms, and one blanket notice would be wrong for part of it.

The rows that say *with the date of the version* are sources whose terms ask for
the date of the data beside the notice. A ride of theirs carries that date in
`dataSourceDate` (`YYYY-MM-DD`) beside its `dataSource`, and the answer's
`attribution` — like the note in a map image — names the source with it:
`<notice> – YYYY-MM-DD`.

## Support

<support@projektionisten.de>

<!-- legal:begin (generated) -->

[Imprint](https://www.projektionisten.de/impressum) · [Privacy policy](https://www.projektionisten.de/mcp/mcp-datenschutz) (German) · [Terms of use](https://www.projektionisten.de/mcp/mcp-nutzungsbedingungen) (German)

<!-- legal:end -->
