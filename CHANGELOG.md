# Changelog

What changed about the tools of the two servers, newest first. One section per
server that moved, and the version number beside its name says what kind of
change it was:

- **major** — a tool was removed or renamed, an argument was removed, an argument
  changed its type, or an argument became mandatory. A call that worked yesterday
  can stop working.
- **minor** — a tool or an optional argument was added. Nothing that worked stops.
- **patch** — only the wording changed.

The entries are generated from what the servers answer to `tools/list`; what a
tool takes and returns in detail is in the `tools.md` beside it.

The first entry records a change made before this file existed, and therefore
carries the version the two servers were already published under. From the next
entry on, the number moves with the rule above.

## 2026-09-16 — MobilityMCP 0.1.0

- arguments changed: `search_place`, `reverse_geocode`

## 2026-09-16 — TourismMCP 0.1.0

- removed: `get_current_air_quality`, `get_air_quality_forecast`
- arguments changed: `search_place`, `reverse_geocode`, `search_tourism`, `get_tourism_details`
- description changed: `get_poi_details`, `expand_kg_pois`, `search_tourism`, `get_tourism_details`, `get_current_weather`
