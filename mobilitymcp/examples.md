# MobilityMCP — example prompts

Written the way somebody actually asks. Each one is followed by what the server
does with it, so you can tell a good answer from a plausible one.

The questions are German because the data is: stop names, line designations and
disruption texts come out of German timetable data and are answered in the
words they are filed under.

## A journey

> Wie komme ich morgen früh um 8 von Hannover Hauptbahnhof nach Braunschweig?

`link_datetime` turns „morgen früh um 8" into an instant, `resolve_location`
turns both station names into ids, and `connections` plans the ride. The answer
names departure and arrival, the lines, and every change.

## What is leaving right now

> Was fährt in den nächsten 20 Minuten von der Haltestelle Aegidientorplatz?

`resolve_location` for the stop, `link_datetime` for „jetzt", then `departures`.
The board comes back with line, destination, platform and the real-time
departure where the source has one.

## Whether something is wrong on the line

> Gibt es gerade Störungen bei der Linie 4 in Hannover?

`disruptions`, narrowed to the line and the area. Two things worth knowing about
the answer: it covers Lower Saxony, Bremen and Hamburg, and it says so in its
own `coverage` field — outside that area an empty answer means *no data*, not
*no disruptions*.

## Where a line runs

> Welche Heide-Shuttle-Linien gibt es, und wo fahren sie entlang?

`line_course` is the one tool of the set that is keyed on a **line** rather than
on a place or a time. It answers the stop sequence and the drawn course, so a
client with a map can put it on one.

## An address you only have as a coordinate

> Ich stehe bei 52.3759, 9.7320 — welche Haltestellen sind in der Nähe?

`nearby` for the stops around the point; `reverse_geocode` if you also want to
know what the point itself is called.

## What it will not do

> Wie wird das Wetter morgen in Hannover?

Nothing here answers that, and the server says so rather than guessing — weather
lives in [TourismMCP](../tourismmcp/README.md).
