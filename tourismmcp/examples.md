# TourismMCP — example prompts

Written the way somebody actually asks. Each one is followed by what the server
does with it, so you can tell a good answer from a plausible one.

The questions are German because the data is: place names, opening hours and
editorial descriptions come out of German sources and are answered in the words
they are filed under.

## A day out, weather permitting

> Was kann ich am Wochenende in Hannover unternehmen, wenn das Wetter mitspielt?

`search_place` turns the city into coordinates, `get_current_weather` or
`get_weather_forecast` reads the conditions, and `expand_kg_pois` or
`search_tourism` returns places. Every place named in the answer came out of one
of those calls — a server that could not reach them says so rather than
remembering a city it has read about.

## Opening hours and what it costs

> Wann hat das Niedersächsische Landesmuseum geöffnet, und was kostet der
> Eintritt?

`resolve_location` or `search_tourism` finds the record, then
`get_tourism_details` for the curated entry or `get_poi_details` for the
OpenStreetMap one. Prices and opening hours are quoted from the record, and
where the record has none, the answer says that instead of estimating.

## Events with a date

> Welche Veranstaltungen gibt es im Juli rund um Cuxhaven?

`search_tourism` with a date range — this is the curated layer, the one with
editorial depth: events with dates, descriptions, media, tours. It is densest in
Lower Saxony; a region outside it answers thinner rather than differently.

## Low tide

> Wann ist morgen Ebbe in Cuxhaven?

`link_datetime` for „morgen", then `get_tide`. The times belong to the **gauge**
the reading came from, and a good answer names it — a gauge a few kilometres
away is a different water level, and attributing its time to the town somebody
asked about would be wrong even when it is close.

## A bike at the station

> Steht gerade ein Leihrad in der Nähe vom Hauptbahnhof Hannover?

`resolve_location` for the station, then `nearby` — the vicinity search reports
sharing offers with the free stock their networks report. It is the one mobility
fact this server carries; journeys and departure boards are
[MobilityMCP](../mobilitymcp/README.md).

## What it will not do

> Wann fährt der nächste Zug nach Bremen?

Nothing here answers that, and the server says so rather than guessing —
timetable service lives in [MobilityMCP](../mobilitymcp/README.md).
