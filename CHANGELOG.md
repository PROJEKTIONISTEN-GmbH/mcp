# Changelog

What changed about the tools of the two servers, newest first. One section per
server that moved, and the version number beside its name says what kind of
change it was:

- **major** — a tool was removed or renamed, an argument was removed, an argument
  changed its type, or an argument became mandatory. A call that worked yesterday
  can stop working.
- **minor** — a tool or an optional argument was added. Nothing that worked stops.
- **patch** — only the wording changed.

The entries are generated from what the servers answer to `tools/list` and from
their registry listing (`server.json`), whose wording counts as a patch; what a
tool takes and returns in detail is in the `tools.md` beside it.

The first entry records a change made before this file existed, and therefore
carries the version the two servers were already published under. From the next
entry on, the number moves with the rule above.

## 2026-10-01 — MobilityMCP 0.6.1

- description changed: `connections`, `departures`, `disruptions`, `line_course`

## 2026-10-01 — TourismMCP 0.1.3

- description changed: `get_poi_details`, `expand_kg_pois`, `search_tourism`, `get_tourism_details`, `get_current_weather`, `get_weather_forecast`, `get_tide`

## 2026-10-01 — MobilityMCP 0.6.0

- arguments changed: `departures`, `show_departures`
- description changed: `departures`

## 2026-10-01 — MobilityMCP 0.5.0

- arguments changed: `connections`, `show_connections`, `show_connections_slim`
- description changed: `connections`

## 2026-09-30 — MobilityMCP 0.4.0

- arguments changed: `connections`, `show_connections`, `show_connections_slim`
- description changed: `connections`

## 2026-09-23 — MobilityMCP 0.3.1

- listing changed: `description`

## 2026-09-23 — TourismMCP 0.1.2

- listing changed: `description`

## 2026-09-22 — TourismMCP 0.1.1

- description changed: `get_poi_details`, `expand_kg_pois`, `search_tourism`, `get_current_weather`, `get_weather_forecast`, `get_tide`, `link_datetime`

## 2026-09-21 — MobilityMCP 0.3.0

- added: `show_connections`, `show_departures`, `show_connections_slim`

## 2026-09-17 — MobilityMCP 0.1.0

- removed: `show_connections`, `show_departures`, `show_connections_slim`

## 2026-09-16 — MobilityMCP 0.1.0

- arguments changed: `search_place`, `reverse_geocode`

## 2026-09-16 — TourismMCP 0.1.0

- removed: `get_current_air_quality`, `get_air_quality_forecast`
- arguments changed: `search_place`, `reverse_geocode`, `search_tourism`, `get_tourism_details`
- description changed: `get_poi_details`, `expand_kg_pois`, `search_tourism`, `get_tourism_details`, `get_current_weather`
