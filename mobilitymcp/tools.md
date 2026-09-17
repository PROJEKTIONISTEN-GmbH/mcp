# MobilityMCP — tools

<!-- tools:begin (generated from the server's tools/list) -->

Generated from the `tools/list` of `https://ai.projektionisten.eu/mmcp` — 13 tools.

Every description below is the one the server sends; it is what your agent reads before it picks a tool. The tool descriptions are German, because the data they answer with is.

## Stadt oder Region suchen — `search_place`

*read-only · not destructive · answers from live outside data*

Löst den NAMEN einer Stadt, Region oder eines Bezirks in Koordinaten und OSM-Ids auf — der erste Schritt, wenn ein GEBIET verortet werden muss („was kann ich in <REGION> unternehmen", „wie ist das Wetter in <CITY>"). **Für eine HALTESTELLE ist dieses Werkzeug fast immer falsch**: es kennt Gebiete, keine Bahnsteige — auf einen Bahnhofs-Namen antwortet es mit dem Stadtteil, und die Id, die es liefert, ist eine OSM-Id und keine fahrbare Halte-Id. **Trägt die Anfrage `Bahnhof`, `Hauptbahnhof`, `Hbf` oder `Bf`, gehört sie an die Ortsauflösung** — „Köln Hauptbahnhof" und „Hannover Bahnhof" also dorthin, nicht hierher: auf das erste antwortet dieses Werkzeug mit einem gleichnamigen Ortsteil (einem in Potsdam), auf das zweite mit der Stadt Hannover. Dasselbe für Adresse und POI — alles, was Start, Ziel oder Abfahrtsort einer Fahrt sein kann; eine Stadt als Fahrt-Endpunkt („von Hannover nach Celle") ebenfalls. Von einer Koordinate zurück zum Namen geht `reverse_geocode`, die Umgebung einer Koordinate listet `nearby`. **Pflicht**: `name`. **Optional**: `lang` — wird für Symmetrie mit den übrigen Geo-Werkzeugen angenommen, derzeit aber nicht ans Backend durchgereicht und ändert das Ergebnis nicht. **Anleitung**: `get_usage_guide` mit `tool='search_place'` — die Abgrenzung im Detail, die typischen Ketten und der Umgang mit einem mehrdeutigen Namen. **Anti-Fab**: nur die zurückgegebenen Namen und Ids nutzen, keine Bauch-Geographie.

| Argument | Required | Type | Description |
|---|---|---|---|
| `format` | no | string | Answer serialisation: `"toon"` (default) or `"json"` — the detail is in `get_usage_guide`. **An agent leaves this out.** |
| `lang` | no | string | Optional ISO language code (e.g. "de", "en"). Currently advisory: the place names are the German ones the data carries, whatever is asked for. Kept in the signature so a multilingual answer needs no new argument. |
| `name` | yes | string | Name of the place to search for. Free-text fuzzy match against the place index behind this endpoint (e.g. "Hannover", "Maschsee", "Wangerland"). |

## Ort auflösen — `resolve_location`

*read-only · not destructive · answers from live outside data*

Löst Freitext in die Id auf, mit der gefahren wird — die Haltestelle, die Adresse oder den POI hinter „von <X> nach <Y>", „erzähl mir was über <POI>". Wo Routing, Abfahrtstafel oder POI-Steckbrief eine Id brauchen, steht es davor — auch wenn der Nutzer die Stadt dazu nennt („Hauptbahnhof Hannover"). Ein GEBIET (Stadt/Region) verortet `search_place`. **Pflicht**: `query` — nur der Name: `'Kelsterbach Bahnhof'`, NICHT `'für Kelsterbach Bahnhof'`. Wegzulassen sind `für`, `vom`, `von`, `nach`, `bis`; ein führendes `am`/`an`/`in`/`zur`/`auf` bleibt stehen — so heißen echte Halte („Am Wehrhahn"). **Optional**: `lat`/`lon` (Ranking-Bias), `limit`, `node_types` (`'stop'`/`'address'`/`'poi'`/`'any'`, mehrere mit Komma in EINEM String; eine andere Art ist ein Argument-Fehler), `city_station` (Query = ganze Stadt → deren (Haupt-)Bahnhof). **Art-Wort in `node_types`, nicht in den Namen**: „Haltestelle X" → `'stop'`, „Adresse X" → `'address'`, „POI X"/„Sehenswürdigkeit X" → `'poi'`, `query` je ohne das Wort. **A→B: zweimal rufen** — Start, Ziel. Jeder Treffer trägt `type` und `location`; NUR ein `stop` hat eine fahrbare DH-Id, nie eine erfinden. **Anleitung**: `get_usage_guide` — Abgrenzung, Argumente, Rangfolge. **Anti-Fab**: nur die Treffer aus dem Output dieses Aufrufs verwenden.

| Argument | Required | Type | Description |
|---|---|---|---|
| `city_station` | no | boolean | When `Some(true)` AND the query is a whole CITY, resolve it to the city's (Haupt-)Bahnhof stop and return ONLY that stop (so „von Hannover nach Celle" routes Bahnhof→Bahnhof). Forwarded to mobility-middleware as `cityStation=true`. A no-op for non-city queries. The orchestrator routing-floor sets this deterministically per endpoint; the model never has to. |
| `format` | no | string | Answer serialisation: `"toon"` (default) or `"json"` — the detail is in `get_usage_guide`. **An agent leaves this out.** |
| `lat` | no | number | Optional latitude (WGS-84) to bias ranking toward nearby stops. |
| `limit` | no | integer | Maximum number of results. Defaults to 5 if omitted. |
| `lon` | no | number | Optional longitude (WGS-84) to bias ranking toward nearby stops. |
| `node_types` | no | string | Which kinds of place to resolve to: `"stop"`, `"address"`, `"poi"` or `"any"`. Several combine in ONE comma-separated string — `"stop,poi"`; an array of the same tokens is read as that string. A kind outside the four is an argument error, not a dropped filter — the shared-mobility kinds among them: a rental station and a taxi rank are not in the place index this searches, they are answered by the radius search `nearby`. Forwarded to mobility-middleware as `journey_node_types`. `"poi"` alone additionally drops non-tourism POI names (Kindergarten, Apotheke, Schule, …) — use this for tourism queries. Nennt der Nutzer die Art selbst, gehört sie hierher statt in `query`: "Haltestelle X" → `"stop"`, "Adresse X" → `"address"`, "POI X"/"Sehenswürdigkeit X" → `"poi"` (und `query` dann ohne das Art-Wort). |
| `query` | yes | string | Free-text query, e.g. "Hauptbahnhof Hannover" or "Linden Markt". Nur der reine Orts-/Haltename — ohne das Wort, das ihn im Satz ankündigt: `"Kelsterbach Bahnhof"`, nicht `"für Kelsterbach Bahnhof"`. Ein führendes `am`/`an`/`in`/`zur`/`auf` bleibt dagegen stehen, weil echte Haltestellen so heißen ("Am Wehrhahn", "In der Au"). |

## Umgebung einer Koordinate — `nearby`

*read-only · not destructive · answers from live outside data*

Findet Haltestellen, POIs und Adressen im Radius um eine KOORDINATE (Schwerpunkt Hannover/Niedersachsen) — „was ist in der Nähe von <lat,lon>", „welche Haltestellen liegen um diesen Punkt". Liegt nur ein Name vor, kommt erst eine Auflösung: `search_place` für eine Stadt oder Region, sonst die Ortsauflösung. **Pflicht**: `latitude`, `longitude`, `radius_m`. **Optional**: `limit` (Default 10); `node_types` — EIN String, mehrere Arten mit Komma: `'stop'`, `'address'`, `'poi'`, die Sharing-Angebote `'bike_rental'`, `'scooter_rental'`, `'car_sharing'`, `'taxi_stand'`, die Abstellanlagen `'park_and_ride'`, `'bike_and_ride'`, oder `'any'` (`'stop,bike_rental'`); `include_mots` (legt die bedienenden Linien über die Haltestellen-Treffer); `only_available` (nur Sharing-Treffer mit gemeldetem freiem Fahrzeug — UNBEKANNTE Verfügbarkeit gilt nicht als frei). **Format**: kompakter Einrück-Text (TOON), kein JSON. **Anleitung**: `get_usage_guide` mit `tool='nearby'` — die Werte im Einzelnen, was `'any'` nicht abdeckt, und was ein Treffer trägt (`category`, `modality`, `parking`, `contactInfo`). **Anti-Fab**: nur die zurückgegebenen Namen, Typen, Distanzen und Verfügbarkeiten nennen; fehlt ein Feld, war es in der Quelle nicht getaggt — Öffnungszeiten und Preise stehen hier NICHT.

| Argument | Required | Type | Description |
|---|---|---|---|
| `format` | no | string | Answer serialisation: `"toon"` (default) or `"json"` — the detail is in `get_usage_guide`. **An agent leaves this out.** |
| `include_mots` | no | boolean | `true` overlays the serving transit lines on stop results. Default `false`, which keeps the answer small. |
| `latitude` | yes | number | Latitude in WGS-84 decimal degrees. |
| `limit` | no | integer | Maximum number of results, at least 1. Default 10. |
| `longitude` | yes | number | Longitude in WGS-84 decimal degrees. |
| `node_types` | no | string | Which kinds of place to answer with: the places the graph holds — `"stop"`, `"address"`, `"poi"` — plus the shared-mobility offers `"bike_rental"`, `"scooter_rental"`, `"car_sharing"` and `"taxi_stand"`, plus the places to leave a vehicle of one's own, `"park_and_ride"` and `"bike_and_ride"`, or `"any"` for all of them. Several combine in ONE comma-separated string — `"stop,bike_rental"`; an array of the same tokens is read as that string. A kind outside the ten is an argument error, not a dropped filter. `"poi"` alone also switches on the non-tourism filter; omit (or `"any"`) for the mixed default, in which each requested kind gets its share of `limit` and the shared-mobility offers share one between them. The two park-and-ride kinds are the exception `"any"` does NOT cover — ask for them by name, and with a `radius_m` of a few kilometres, because such sites are sparse. |
| `only_available` | no | boolean | Set to `true` for „wo kann ich JETZT eines nehmen": of the shared-mobility offers, only those whose live feed reports at least one vehicle ready to be taken are answered with. A station whose availability is unknown is NOT returned then, and neither is a taxi rank, which has no feed to report one. Omit (or `false`) to list every place in range. The other kinds — `"stop"`, `"address"`, `"poi"`, `"park_and_ride"`, `"bike_and_ride"` — are unaffected either way. |
| `radius_m` | yes | number | Search radius in metres. |

## Koordinate benennen — `reverse_geocode`

*read-only · not destructive · answers from live outside data*

Benennt, was an einer KOORDINATE liegt — der Ort („was liegt bei <lat,lon>"), mit `level` eine bestimmte Ebene davon, mit `level='street'` die Straße samt nächster Hausnummer (der Lookup für eine GPS-Startposition). **Abgrenzung**: den Weg zurück (Name → Koordinate und OSM-Ids) geht `search_place`, die fahrbare Halte-Id liefert die Ortsauflösung, und `nearby` listet auf, was UM eine Koordinate liegt, statt den Punkt selbst zu benennen. **Pflicht**: `lat` und `lon` — geschrieben auch `latitude`/`longitude`, so wie `nearby` die Koordinate nimmt; je Aufruf nur eine der beiden Schreibweisen. **Optional**: `radius_m` (Suchradius in METERN; ein Ort, IN dem die Koordinate liegt, hat Abstand 0 und ist in jedem noch so engen Radius dabei — ohne `radius_m` die nächstgelegenen Treffer), `limit` (Höchstzahl Treffer, Default 10, Maximum 50) und `level` — `'place'` (Default: die ganze Ortshierarchie, feinste Ebene zuerst), `'city'` (die Stadt/Gemeinde), `'suburb'` (der Stadtteil) oder `'street'`. Kennt der Datensatz die gewünschte Ebene hier nicht, antwortet die nächst-gröbere, erkennbar am `place_type`; ein unbekannter Wert wirkt wie `'place'`. **Anleitung**: `get_usage_guide` mit `tool='reverse_geocode'`. **Anti-Fab**: nur die zurückgegebenen Orts- und Straßen-Namen verwenden, einschließlich der Hausnummer aus dem Datensatz — nie eine erfinden.

| Argument | Required | Type | Description |
|---|---|---|---|
| `format` | no | string | Answer serialisation: `"toon"` (default) or `"json"` — the detail is in `get_usage_guide`. **An agent leaves this out.** |
| `lat` | yes | number | Latitude of the coordinate to reverse-geocode. Also accepted spelled `latitude`, the way the vicinity search takes it — one of the two spellings per call. |
| `level` | no | string | Optional resolution level. `"place"` (default) answers with the coordinate's whole admin hierarchy, finest first (suburb → town → county → state → country); `"city"` with the town/municipality it lies in; `"suburb"` with the suburb; `"street"` with the nearest STREET/ADDRESS (street name + nearest house number, e.g. `"Münzstraße 3-4"`) — the level needed to label a real GPS start position. Where the data has no place at the requested level, the next coarser one answers, recognisable by its `place_type`. Any other / omitted value behaves as `"place"`. |
| `limit` | no | integer | Optional maximum number of hits, nearest first. Default 10, upper bound 50; a larger value is served as 50 and `0` as the default. Applies to every level and with or without `radius_m`. |
| `lon` | yes | number | Longitude of the coordinate to reverse-geocode. Also accepted spelled `longitude`, the way the vicinity search takes it — one of the two spellings per call. |
| `radius_m` | no | number | Optional radius in METRES. When set, returns the places inside the circle, sorted nearest-first and capped to `limit`. A place the coordinate LIES INSIDE is at distance 0 and is therefore in every radius, however tight. When omitted, returns the `limit` nearest places. |

## Verbindung suchen — `connections`

*read-only · not destructive · answers from live outside data*

Plant ÖPNV-Verbindungen von A nach B, deutschlandweit, Fernrouten eingeschlossen — **als Daten, für Ergebnisse, die du weiterverarbeitest**. Steht `show_connections` in deinem Werkzeug-Satz, ist das der Vordereingang für die Frage eines Menschen. Für „wie komme ich von X nach Y" kommt die Auskunft aus diesen Werkzeugen, nicht aus dem Allgemeinwissen. **Pflicht sind BEIDE Endpunkte**: `origin_id` UND `destination_id` aus einer Ortsauflösung, mit `origin_type`/`destination_type` — ODER `origin_lat`+`origin_lon`+`destination_lat`+`destination_lon`; ein NAME in einem Id-Feld ist ein Argument-Fehler. **Optional**: `time`, `is_arrival_time`; `via_id` bzw. `via_lat`+`via_lon` mit `via_type`; die hart einschränkenden Filter `modes`, `only_lines`/`exclude_lines`, `submodes_allow`/`submodes_deny`, `max_transfers`, `max_walk_minutes`/`max_walk_meters`, `walk_speed`, `accessibility_profile`, `include_rental_bike`, `prefer_flat`, `mobility_profile`; `verbosity`, `limit`, `render_payload`. **Höhenmeter**: nur Fuß-/Rad-Abschnitte tragen `ascentInMeters`/`descentInMeters`. **Durchbindung**: `staySeated: true` an einem Abschnitt heißt, der Fahrgast bleibt aus dem vorigen sitzen — kein Umstieg, also auch keinen zählen. Fehlt das Feld, ist nichts gesagt; nie selbst herleiten. **Rad-Tempo**: Bei einer Radfahrt setzt `cycling_profile` das Tempo des Fahrers und damit die gemeldete Dauer — `family` bei kleinen Kindern, `ebike` bei E-Bike/Pedelec; sonst weglassen (= `normal`). **Fahrradmitnahme**: Geht es um ein Rad, sprich je Abschnitt `bikeCarriageConditions` aus (Bedingungen des Verbunds zur Abfahrtszeit); `bikesAllowed`/`bikeCarriageNote` sind nur die Grundregel, `bikeCarriageCoverage` die Wissensgrenze. **Genannter Wunsch**: Nennt jemand, was ihn stört ("nicht in der Innenstadt parken"), hol die Option, die das löst, als EIGENEN Aufruf (`mobility_profile`) und empfiehl sie aus ihrem Nutzen — mit dem Nachteil, den die Antwort ihr gibt. **Anleitung**: `get_usage_guide` mit `tool='connections'` — lies sie, bevor du filterst. **Anti-Fab**: Linien, Zeiten, Halte, Anlagen und Gleise ausschließlich aus dem Output dieses Aufrufs.

| Argument | Required | Type | Description |
|---|---|---|---|
| `accessibility_profile` | no | string | Mobility profile by name: `"Standard"` (default), `"WheelchairRobust"`, `"WalkerComfort"` or `"IndividualComfort"`. Each bundles its accessibility constraints and walk speed; another name is an argument error. The two wheelchair profiles find no ride at all today — the answer then carries `accessibility_routing` with `status: no_data` beside ordinary connections, which are to be reported as ordinary ones. What each profile constrains is in the usage guide. |
| `cycling_profile` | no | string | Wer auf dem Rad sitzt, und damit das Tempo der Rad-Abschnitte: `family` (mit kleinen Kindern), `normal` (Default, unverändert) oder `ebike` (Pedelec). Ein anderer Name ist ein Argument-Fehler. |
| `cycling_speed` | no | number | Rad-Tempo in Metern/Sekunde, 1 bis 12 — außerhalb ein Argument-Fehler. Überschreibt `cycling_profile`. Richtwerte: Familie ≈2.8, normal ≈5, Pedelec ≈6.5. |
| `destination_id` | no | string | Destination id. Same form as `origin_id`. |
| `destination_lat` | no | number | Destination latitude. Same use as `origin_lat`. |
| `destination_lon` | no | number | Destination longitude. Goes with `destination_lat`. |
| `destination_type` | no | string | What `destination_id` is. Same values as `origin_type`. |
| `exclude_lines` | no | array | Line blacklist, same token form as `only_lines`. A journey is dropped if any transit leg uses one of them. |
| `extended` | no | boolean | `true` keeps the map data in the text: every stop's coordinate, every leg's polyline, and the whole stop sequence with them — for an application that DRAWS the route, at two to three times the answer. A chat client leaves it off. |
| `format` | no | string | Answer serialisation: `"toon"` (default) or `"json"` — the detail is in `get_usage_guide`. **An agent leaves this out.** |
| `include_rental_bike` | no | boolean | `true` adds a rental-bike leg (park-and-bike / station rental). Equivalent to naming `"bike"` in `modes` together with a transit mode. |
| `is_arrival_time` | no | boolean | `true` reads `time` as the desired ARRIVAL time instead of the departure time. |
| `limit` | no | integer | Maximum number of journeys, at least 1. Default 5, values above 15 are served as 15. |
| `max_transfers` | no | integer | Upper bound on interchanges: `0` = direct, up to `3`. Omit unless the user constrains transfers: the backend then derives the search depth from the trip's air-line distance and escalates it when the first attempt finds nothing, so omitting never loses a connection. |
| `max_walk_meters` | no | integer | Cap on the summed walking DISTANCE in metres. Works like `max_walk_minutes`, on the metres walked. |
| `max_walk_minutes` | no | integer | Cap on the summed walking TIME in minutes. A post-filter drops journeys above it; if that leaves nothing, the backend searches once more avoiding footpaths before the answer is empty. |
| `mobility_profile` | no | string | How the first and the last mile may be covered, by name: `"transit"` (default — both ends on foot), `"transit_plus_sharing"` (a shared bike), `"door_to_door"` (being driven), `"park_and_ride"` or `"bike_and_ride"` (one's OWN car or bicycle, left where the transit leg starts). Another name is an argument error. It widens those two ends, restricts no ride and promises nothing. When to set which, and how it composes with `modes`, is in the usage guide. |
| `modes` | no | array | Transport-mode restriction, not a preference: nothing outside the list is planned. The footpath always stays allowed — the way to, from and between stops goes on foot — while `["foot"]` on its own plans no ride at all. Entries: `"bike"`, `"bike_rental"`, `"foot"`, `"car"`, `"taxi"`, `"scooter"`, `"transit"`, `"bus"`, `"tram"`, `"rail"`, `"subway"`, `"ferry"`; several combine. An entry outside that list is an argument error, not a dropped filter. Omitted or empty = all standard transit modes, the ferry included. Set only when the user insists on a mode; the guide has the vocabulary and the traps. |
| `only_lines` | no | array | Line whitelist. Entries are line tokens as printed on the vehicle (`"6"`, `"U3"`); a journey is kept only if every transit leg uses one of them. |
| `origin_id` | no | string | Origin id. A DHID (`de:NNNNN:NNN`) for a stop, otherwise the resolver's `main` id of the address or POI. Resolve the place first — a name here is an argument error. |
| `origin_lat` | no | number | Origin latitude, WGS-84 decimal degrees — the fallback when no id is at hand, and the one retry worth making when resolved ids find nothing. |
| `origin_lon` | no | number | Origin longitude. Goes with `origin_lat`. |
| `origin_type` | no | string | What `origin_id` is: `"stop"` (default), `"address"` or `"poi"`. A stop id is auto-prefixed `GTFS.de:`; the other two are forwarded verbatim so the backend resolves the real name. |
| `prefer_flat` | no | boolean | `true` plant die hügel-ärmste statt der sonst gewählten Rad-Route: die Suche gewichtet die Steigung und nimmt dafür Umwege in Kauf, nur auf Rad-Abschnitten. Keine Zusage — ohne flachere Alternative kommt dieselbe Route; die Höhenmeter der Antwort sagen es. Wann setzen: die Anleitung. |
| `render_payload` | no | boolean | `true` appends the same journeys again as raw JSON, addressed to the user rather than to the model — the map geometry an application draws from. A chat client leaves this off: it is the whole answer twice. |
| `submodes_allow` | no | array | Fine submode whitelist — finer than `modes`, which only separates bus/tram/rail/subway. Entries: `"sbahn"`, `"regionalbahn"`, `"ice"`, `"ic"`, `"ir"`, `"nj"`, `"fernverkehr"`, `"stadtbahn"`, `"ubahn"`, `"bus"`, `"regionalbus"`, `"stadtbus"`. A journey is kept only if every transit leg matches one. |
| `submodes_deny` | no | array | Fine submode blacklist, same vocabulary as `submodes_allow`. A journey is dropped if any transit leg matches one. |
| `time` | no | string | ISO 8601 timestamp (e.g. `"2026-06-15T08:30:00Z"`). Omit to plan from now. Resolve a spoken time with the time tool rather than computing one. |
| `verbosity` | no | string | How much of each journey comes back: `"compact"` (default — everything an answer is cited from) or `"full"` (adds the stops in between). A third word is an argument error, not the default. |
| `via_id` | no | string | Intermediate stop the route must pass through („über X"), as an id in the same form as `origin_id`. The route is then planned A → via → B. |
| `via_lat` | no | number | Via latitude, for an intermediate address or POI without an id. Goes with `via_lon`. |
| `via_lon` | no | number | Via longitude. Goes with `via_lat`. |
| `via_type` | no | string | What `via_id` is. Same values as `origin_type`. |
| `walk_speed` | no | number | Walking speed of the foot legs in metres/second, between 0.3 and 3 m/s — outside that it is an argument error. Default ≈1.33, slow ≈0.7, brisk ≈2.0. Overrides the walk speed of `accessibility_profile`. |

## Abfahrtstafel lesen — `departures`

*read-only · not destructive · answers from live outside data*

Abfahrten oder Ankünfte für EINE Haltestelle — **als Daten, für Ergebnisse, die du weiterverarbeitest**. Steht `show_departures` in deinem Werkzeug-Satz, ist das der Vordereingang für die Frage eines Menschen. Für „wann fährt von X", „letzte Stadtbahn von X" kommt die Auskunft aus diesen Werkzeugen, nicht aus dem Allgemeinwissen; für eine A→B-Frage aus `connections`. **Pflicht** ist eine konkrete Stop-`id` aus einer vorhergehenden Ortsauflösung (z.B. `'de:03241:57'`); ein Haltestellen-NAME in `id` ist ein Argument-Fehler, keine leere Tafel. **Optional**: `time` und `is_arrival_time`; `modes`; `direction` (Halt-Name, in dessen Richtung gefahren wird) bzw. für eine weiche Angabe `target_bearing` mit `bearing_tolerance`; `arrival_departure`; für „letzte Fahrt des Tages" `time_range_seconds` zusammen mit `last_of_window`; und für den Umfang `verbosity`, `limit` sowie `render_payload`. `modes` und `direction` schränken hart ein. **Eine leere Tafel ist keine Auskunft**: blieb sie mit `['tram']` („Straßenbahn") leer, mit `['tram','subway']` nachfassen — sonst ohne den Filter. **Anleitung**: `get_usage_guide` mit `tool='departures'` — die Fenster-Kette für „letzte Fahrt", die Echtzeit-, Ausfall- und Zitier-Felder. **Anti-Fab**: Linien, Richtungen, Zeiten, Gleise, Verspätung und Ausfall ausschließlich aus dem Output dieses Aufrufs.

| Argument | Required | Type | Description |
|---|---|---|---|
| `arrival_departure` | no | string | `"arrivals"` (when vehicles reach `id`) or `"departures"` (default) — a third value is an argument error. Wins over `is_arrival_time` when both are set. |
| `bearing_tolerance` | no | number | Angular tolerance in degrees for `target_bearing`. Default ±60°. |
| `direction` | no | string | Name of a stop the user wants to head TOWARD („Richtung Wettbergen") — not a routing destination. The board keeps only departures heading that way, whether the named stop is the terminus or a stop downstream of the queried one. Matching is case-insensitive and area-prefix-tolerant. |
| `extended` | no | boolean | `true` keeps the map data in the text: every stop's coordinate, the whole course of each run, and a polyline where the board carries one — for an application that DRAWS the board. It multiplies the answer. A chat client leaves it off. |
| `format` | no | string | Answer serialisation: `"toon"` (default) or `"json"` — the detail is in `get_usage_guide`. **An agent leaves this out.** |
| `id` | yes | string | Stop id to read the board of, as a DHID (`"de:03241:57"`). Resolve the stop first — a name here is an argument error. |
| `is_arrival_time` | no | boolean | `true` reads `time` as an ARRIVAL time — the same as `arrival_departure = "arrivals"`. |
| `last_of_window` | no | boolean | Which end of a `time_range_seconds` window to read: omitted or `false` the FIRST departures, `true` the LAST ones (the end-of-service question). An answer carrying `window_truncated: true` beside its board did NOT reach the window's end — its last entry is then not the last departure. Without a window it has no effect. |
| `limit` | no | integer | Maximum number of departures, at least 1. Default 5, values above 15 are served as 15. |
| `modes` | no | array | Transport-mode filter; the board then shows nothing else. Entries: `"bus"`, `"tram"`, `"subway"`, `"train"`/`"rail"`, `"ferry"`, `"transit"`; several combine. Another entry — a street mode, a misspelling — is an argument error, not a dropped filter: a board lists scheduled services. Omitted or empty = all modes. A Stadtbahn or light-rail mention needs BOTH labels — `["tram","subway"]`: which of the two such a network carries depends on the dataset rather than on the vehicle. Set the filter only when the user really asks for a vehicle type; the guide has the rest of the vocabulary and the traps. |
| `render_payload` | no | boolean | `true` appends the same board again as raw JSON, addressed to the user rather than to the model — for an application that draws a departure board. A chat client leaves this off: it is the whole answer twice. |
| `target_bearing` | no | number | Target bearing in degrees (0=N, 90=E, 180=S, 270=W) for a SOFT direction („Richtung Norden", „Richtung Stadt") instead of a stop name. The board keeps only departures whose course lies within `bearing_tolerance` of it. Mutually exclusive with `direction`. |
| `time` | no | string | ISO 8601 timestamp (e.g. `"2026-06-15T08:30:00Z"`). Omit for now. Resolve a spoken time with the time tool rather than computing one. |
| `time_range_seconds` | no | integer | Time WINDOW in seconds, starting at `time`. Together with `last_of_window` this answers „letzte Fahrt": pick it so the window ENDS after end of service (from the evening on, `28800` = 8 h rather than 4 h). Omit for a normal next-N lookup. |
| `verbosity` | no | string | How much of each departure comes back: `"compact"` (default — everything an answer is cited from) or `"full"` (adds the onward stops of the run, and with them the per-stop cancellation marker). A third word is an argument error, not the default. |

## Störungen abfragen — `disruptions`

*read-only · not destructive · answers from live outside data*

Aktuelle Störungen, Baustellen und Liniensperrungen im ÖPNV. Für „gibt es Störungen", „Ausfälle in X" ist dieses Werkzeug die Auskunft, nicht das Allgemeinwissen. **Datenlage — wichtig**: Störungsmeldungen liegen derzeit für Niedersachsen, Bremen und Hamburg vor; außerhalb gibt es dafür KEINE Datengrundlage. (Verbindungen und Abfahrten sind davon nicht betroffen — die sind deutschlandweit.) Eine leere Antwort ist deshalb nur dann eine Entwarnung, wenn das Feld `coverage` sie als solche ausweist; außerhalb der abgedeckten Gebiete heißt sie, dass du es nicht weißt. **Keine Pflicht-Args** — ohne Filter kommen die aktiven Meldungen, gekappt auf `limit`. Optional: `line` (Liniennummer/-token, z.B. `'6'`, `'U3'`) und `stop` (Haltestellen-Name) verknüpfen UND; `area` (Stadt/Gemeinde) bestimmt das `coverage` und sortiert die unverorteten Meldungen — es unterdrückt sie nicht: die kommen getrennt in `unplaced` zurück, auch ohne `area`. Nenne nur Filter, die der Nutzer wirklich nannte. **Format**: kompakter Einrück-Text (TOON), kein JSON; die Schreibweise steht in der Anleitung. **Anleitung**: `get_usage_guide` mit `tool='disruptions'` — die sechs `coverage`-Werte, die beiden Listen, die Felder einer Meldung und woran eine gekappte Antwort zu erkennen ist. **Anti-Fab**: nur zurückgegebene Meldungen nennen; eine Meldung ohne betroffene Linie und ohne betroffenen Halt ist eine allgemeine Störung und wird keiner Linie zugeschrieben, eine Meldung aus `unplaced` keinem Ort.

| Argument | Required | Type | Description |
|---|---|---|---|
| `area` | no | string | Area or city name (`"Hannover"`). Matches the municipality the affected stops lie in, and decides the `coverage` the answer reports. It never suppresses a report: the ones it cannot place come back in `unplaced`. |
| `format` | no | string | Answer serialisation: `"toon"` (default) or `"json"` — the detail is in `get_usage_guide`. **An agent leaves this out.** |
| `include_unplaced` | no | boolean | `false` leaves the `unplaced` list out of the answer (default `true`). It only shortens the answer — `coverage` still counts what was left out, so it never turns into an all-clear. |
| `limit` | no | integer | Maximum number of disruptions, at least 1. Default 5, values above 15 are served as 15. It shortens the answer, it does not choose better entries — an answer that had to leave something out says so (`truncated` + `total`). |
| `line` | no | string | Line token as printed on the vehicle (`"6"`, `"169"`, `"U3"`). A disruption is kept only if it names that line — under one of its `affected_lines` or one of its `affected_line_refs`. |
| `stop` | no | string | Stop name (`"Kröpcke"`). Case-insensitive substring match against the disruption's `affected_stops`. |

## Linienverlauf abfragen — `line_course`

*read-only · not destructive · answers from live outside data*

Linienverlauf: die Halte-Folge einer ÖPNV-LINIE und ihre gezeichnete Strecke für die Karte — „wo fährt die Y entlang", „welche X-Linien gibt es". Für Linien-Fragen: diese Auskunft, nicht das Allgemeinwissen. **Pflicht ist EINES**: `query` — Name/Nummer, wie ein Mensch sie schreibt (`'Heide-Shuttle'`, `'HS 2'`); ODER `id` aus einer früheren Antwort = GENAU EINE Linie. Sonst frag nach. **Optional**: `area` — der Ort, in dem die Linie fährt (`'Hannover'`; Dutzende Linien heißen `'100'`), `limit` (Default 8), `render_payload`, `format`. **Die Antwort**: je Fahrtrichtung ein Verlauf mit `headsign`, den `halts` als Namen und `pathOnMap` — der Strecke im Text, auf max. ~50 m vereinfacht; volle Auflösung: `render_payload: true`. **Mehrdeutig**: meint die Frage mehr Linien als eine Antwort ganz liest, kommt statt `lines` die Liste `candidates` (Nummer, Betreiber, Id) — ungezeichnet; dann den Ort erfragen oder mit `id` nachfassen. **Grenzen**: `geometryDegraded: true` heißt Luftlinien-Kette statt Straßenverlauf, `note` sagt es. Je Richtung EIN Fahrtmuster; ist `patternsTotal` größer, fährt sie üblicherweise so. `matchedTotal` über der Zahl der Linien heißt Ausschnitt. Unbekannter Ort: `note` sagt es, nichts wurde eingegrenzt. **Nicht dafür**: A→B (`connections`), Abfahrten (`departures`), Störungen (`disruptions`). **Anleitung**: `get_usage_guide` mit `tool='line_course'`. **Anti-Fab**: Linien, Halte, Reihenfolge, Strecke nur aus diesem Aufruf; findet sich keine, sag das.

| Argument | Required | Type | Description |
|---|---|---|---|
| `area` | no | string | The town or city the line runs in (`"Hannover"`), where the user named one — the other way to make a line number mean ONE line. Beside `query`; no effect beside `id`, which already means one line. |
| `format` | no | string | Answer serialisation: `"toon"` (default) or `"json"` — the detail is in `get_usage_guide`. **An agent leaves this out.** |
| `id` | no | string | A line id out of an earlier answer (`"de:VBN-VH:6002:_3"`) — the one way to name exactly one line when a name means several. |
| `limit` | no | integer | Maximum number of lines, at least 1. Default 8, values above 15 are served as 15. Ignored beside `id`, which is one line. |
| `query` | no | string | Line name or number, as written on the vehicle or in the timetable (`"Heide-Shuttle"`, `"HS 2"`, `"U3"`). Matched against the beginning of both, without case and without separators. |
| `render_payload` | no | boolean | `true` appends the whole answer again, addressed to the user rather than to the model: the halts with their coordinates, and every course's line at the resolution it was drawn at rather than the simplified `pathOnMap` the text carries. For an application that needs the exact line; a client that draws the answer's own `pathOnMap` leaves this off. |

## Zeitangabe auflösen — `link_datetime`

*read-only · not destructive · answers from live outside data*

Löst eine Zeit-Nennung in einen absoluten ISO-8601-Zeitpunkt auf — „morgen früh", „heute Abend um 18 Uhr", „in zwei Stunden", „um 8:30". Auch „jetzt"/„now"/„aktuell" wird aufgelöst: nutze das als kanonische Quelle für die aktuelle Zeit, statt dir selbst ein Datum auszudenken. Das Ergebnis (`candidates[].datetime_range.start`) geht direkt als `connections.time` bzw. `departures.time` weiter. Nennt der Nutzer bereits eine vollständige ISO-8601-Zeit mit Offset, ist kein Aufruf nötig. **Pflicht**: `text` — die Nennung, so wie der Nutzer sie schrieb. **Optional**: `lang` (`'de'` Default, `'en'`, `'fr'`) und `tz` (IANA-Name; ohne ihn wird die Lokalzeit als `Europe/Berlin` gelesen und als korrekter UTC-Instant zurückgegeben). **Was zurückkommt**: EIN Zeitpunkt, kein Fenster — `start` und `end` sind derselbe Instant. Eine nicht auflösbare Phrase liefert `candidates: []`; dann nachfragen, statt selbst zu rechnen. **Anleitung**: `get_usage_guide` mit `tool='link_datetime'` — insbesondere, auf welche Stunde eine vage Tageszeit fällt und was dann zu tun ist. **Anti-Fab**: Datum und Uhrzeit sind Fakten wie eine Liniennummer und stammen aus diesem Werkzeug — kein selbst-erfundenes Datum, keine eigene Datums-Arithmetik. Returns `candidates[].datetime_range:{start,end}`.

| Argument | Required | Type | Description |
|---|---|---|---|
| `format` | no | string | Answer serialisation: `"toon"` (default) or `"json"` — the detail is in `get_usage_guide`. **An agent leaves this out.** |
| `lang` | no | string | Optional ISO language code (default `de`). Supported: `de`, `en`, `fr`. |
| `text` | yes | string | Free-text time mention to resolve via the rustling NLU. E.g. "morgen früh", "heute Abend um 18 Uhr", "in zwei Stunden". |
| `tz` | no | string | Optional IANA timezone the stated local time is interpreted in (default `Europe/Berlin`). DST-aware. Pass e.g. `"Europe/Berlin"`; omit for the default. Determines the UTC instant emitted for a wall-clock mention ("8:30" → `06:30Z` in summer). |

## Anleitung lesen — `get_usage_guide`

*read-only · not destructive · answers from live outside data*

Die ausführliche Anleitung zu den Werkzeugen dieses Katalogs: wofür ein Werkzeug da ist, wogegen es abzugrenzen ist, was seine Argumente bewirken, was zurückkommt und was daraus zitiert werden darf. Die `description` eines Werkzeugs ist die Kurzform, dieser Text die vollständige. **Optional** `tool` — der Name genau eines Werkzeugs, dessen Abschnitt du lesen willst; weggelassen kommt die ganze Anleitung. Lies den Abschnitt eines Werkzeugs, bevor du dessen Filter setzt oder ein leeres Ergebnis als Antwort weitergibst. **Der Abruf ohne `tool` ist teuer**: er bringt die Abschnitte ALLER Werkzeuge dieses Katalogs auf einmal, ein Vielfaches eines einzelnen. Setze `tool`, sobald feststeht, um welches Werkzeug es geht; ohne Argument nur für den Überblick über den ganzen Katalog.

| Argument | Required | Type | Description |
|---|---|---|---|
| `tool` | no | string | Der Name genau eines Werkzeugs aus diesem Katalog, dessen Abschnitt zurückkommen soll. Weglassen für die ganze Anleitung. |

## Verbindung anzeigen — `show_connections`

*read-only · not destructive · answers from live outside data*

**Der Vordereingang für jede Verbindungsfrage eines Menschen**, auch wenn er nur nach Zeiten fragt: plant die Fahrt wie `connections` und liefert sie zusätzlich als Ansicht aus, die ein Wirt mit Ansichten dem Nutzer zeigt — je Verbindung eine Karte mit Zeiten, Dauer, Umstiegen und Echtzeit-Lage. **Beantwortest du einem Menschen eine Verbindungsfrage, ruf dies statt `connections`**, das für Ergebnisse ist, die du weiterverarbeitest. **Pflicht sind BEIDE Endpunkte**, wie dort: `origin_id` UND `destination_id` — oder die vier Koordinaten-Felder; `render_payload` entfällt. Der Antwort-Text ist der von `connections`; die Rohdaten daneben tragen nur die Felder, die die Ansicht zeichnet, ihr Kartenverlauf davon im `_meta`. **Anleitung**: `get_usage_guide` mit `tool='connections'`. **Anti-Fab**: Linien, Zeiten, Halte und Gleise ausschließlich aus dem Output dieses Aufrufs.

| Argument | Required | Type | Description |
|---|---|---|---|
| `accessibility_profile` | no | string | Mobility profile by name: `"Standard"` (default), `"WheelchairRobust"`, `"WalkerComfort"` or `"IndividualComfort"`. Each bundles its accessibility constraints and walk speed; another name is an argument error. The two wheelchair profiles find no ride at all today — the answer then carries `accessibility_routing` with `status: no_data` beside ordinary connections, which are to be reported as ordinary ones. What each profile constrains is in the usage guide. |
| `cycling_profile` | no | string | Wer auf dem Rad sitzt, und damit das Tempo der Rad-Abschnitte: `family` (mit kleinen Kindern), `normal` (Default, unverändert) oder `ebike` (Pedelec). Ein anderer Name ist ein Argument-Fehler. |
| `cycling_speed` | no | number | Rad-Tempo in Metern/Sekunde, 1 bis 12 — außerhalb ein Argument-Fehler. Überschreibt `cycling_profile`. Richtwerte: Familie ≈2.8, normal ≈5, Pedelec ≈6.5. |
| `destination_id` | no | string | Destination id. Same form as `origin_id`. |
| `destination_lat` | no | number | Destination latitude. Same use as `origin_lat`. |
| `destination_lon` | no | number | Destination longitude. Goes with `destination_lat`. |
| `destination_type` | no | string | What `destination_id` is. Same values as `origin_type`. |
| `exclude_lines` | no | array | Line blacklist, same token form as `only_lines`. A journey is dropped if any transit leg uses one of them. |
| `extended` | no | boolean | `true` keeps the map data in the text: every stop's coordinate, every leg's polyline, and the whole stop sequence with them — for an application that DRAWS the route, at two to three times the answer. A chat client leaves it off. |
| `format` | no | string | Answer serialisation: `"toon"` (default) or `"json"` — the detail is in `get_usage_guide`. **An agent leaves this out.** |
| `include_rental_bike` | no | boolean | `true` adds a rental-bike leg (park-and-bike / station rental). Equivalent to naming `"bike"` in `modes` together with a transit mode. |
| `is_arrival_time` | no | boolean | `true` reads `time` as the desired ARRIVAL time instead of the departure time. |
| `limit` | no | integer | Maximum number of journeys, at least 1. Default 5, values above 15 are served as 15. |
| `max_transfers` | no | integer | Upper bound on interchanges: `0` = direct, up to `3`. Omit unless the user constrains transfers: the backend then derives the search depth from the trip's air-line distance and escalates it when the first attempt finds nothing, so omitting never loses a connection. |
| `max_walk_meters` | no | integer | Cap on the summed walking DISTANCE in metres. Works like `max_walk_minutes`, on the metres walked. |
| `max_walk_minutes` | no | integer | Cap on the summed walking TIME in minutes. A post-filter drops journeys above it; if that leaves nothing, the backend searches once more avoiding footpaths before the answer is empty. |
| `mobility_profile` | no | string | How the first and the last mile may be covered, by name: `"transit"` (default — both ends on foot), `"transit_plus_sharing"` (a shared bike), `"door_to_door"` (being driven), `"park_and_ride"` or `"bike_and_ride"` (one's OWN car or bicycle, left where the transit leg starts). Another name is an argument error. It widens those two ends, restricts no ride and promises nothing. When to set which, and how it composes with `modes`, is in the usage guide. |
| `modes` | no | array | Transport-mode restriction, not a preference: nothing outside the list is planned. The footpath always stays allowed — the way to, from and between stops goes on foot — while `["foot"]` on its own plans no ride at all. Entries: `"bike"`, `"bike_rental"`, `"foot"`, `"car"`, `"taxi"`, `"scooter"`, `"transit"`, `"bus"`, `"tram"`, `"rail"`, `"subway"`, `"ferry"`; several combine. An entry outside that list is an argument error, not a dropped filter. Omitted or empty = all standard transit modes, the ferry included. Set only when the user insists on a mode; the guide has the vocabulary and the traps. |
| `only_lines` | no | array | Line whitelist. Entries are line tokens as printed on the vehicle (`"6"`, `"U3"`); a journey is kept only if every transit leg uses one of them. |
| `origin_id` | no | string | Origin id. A DHID (`de:NNNNN:NNN`) for a stop, otherwise the resolver's `main` id of the address or POI. Resolve the place first — a name here is an argument error. |
| `origin_lat` | no | number | Origin latitude, WGS-84 decimal degrees — the fallback when no id is at hand, and the one retry worth making when resolved ids find nothing. |
| `origin_lon` | no | number | Origin longitude. Goes with `origin_lat`. |
| `origin_type` | no | string | What `origin_id` is: `"stop"` (default), `"address"` or `"poi"`. A stop id is auto-prefixed `GTFS.de:`; the other two are forwarded verbatim so the backend resolves the real name. |
| `prefer_flat` | no | boolean | `true` plant die hügel-ärmste statt der sonst gewählten Rad-Route: die Suche gewichtet die Steigung und nimmt dafür Umwege in Kauf, nur auf Rad-Abschnitten. Keine Zusage — ohne flachere Alternative kommt dieselbe Route; die Höhenmeter der Antwort sagen es. Wann setzen: die Anleitung. |
| `submodes_allow` | no | array | Fine submode whitelist — finer than `modes`, which only separates bus/tram/rail/subway. Entries: `"sbahn"`, `"regionalbahn"`, `"ice"`, `"ic"`, `"ir"`, `"nj"`, `"fernverkehr"`, `"stadtbahn"`, `"ubahn"`, `"bus"`, `"regionalbus"`, `"stadtbus"`. A journey is kept only if every transit leg matches one. |
| `submodes_deny` | no | array | Fine submode blacklist, same vocabulary as `submodes_allow`. A journey is dropped if any transit leg matches one. |
| `time` | no | string | ISO 8601 timestamp (e.g. `"2026-06-15T08:30:00Z"`). Omit to plan from now. Resolve a spoken time with the time tool rather than computing one. |
| `verbosity` | no | string | How much of each journey comes back: `"compact"` (default — everything an answer is cited from) or `"full"` (adds the stops in between). A third word is an argument error, not the default. |
| `via_id` | no | string | Intermediate stop the route must pass through („über X"), as an id in the same form as `origin_id`. The route is then planned A → via → B. |
| `via_lat` | no | number | Via latitude, for an intermediate address or POI without an id. Goes with `via_lon`. |
| `via_lon` | no | number | Via longitude. Goes with `via_lat`. |
| `via_type` | no | string | What `via_id` is. Same values as `origin_type`. |
| `walk_speed` | no | number | Walking speed of the foot legs in metres/second, between 0.3 and 3 m/s — outside that it is an argument error. Default ≈1.33, slow ≈0.7, brisk ≈2.0. Overrides the walk speed of `accessibility_profile`. |

## Abfahrtstafel anzeigen — `show_departures`

*read-only · not destructive · answers from live outside data*

**Der Vordereingang für jede Abfahrtsfrage eines Menschen**: liest die Tafel einer Haltestelle wie `departures` und liefert sie zusätzlich als Ansicht aus, die ein Wirt mit Ansichten dem Nutzer zeigt — je Abfahrt Linie, Ziel, Gleis/Steig, Zeit und Echtzeit-Lage. **Beantwortest du einem Menschen eine Abfahrtsfrage, ruf dies statt `departures`**, das für Ergebnisse ist, die du weiterverarbeitest. **Pflicht ist die Halte-Id**, wie dort: `id` — eine Id aus `resolve_location`, kein Haltestellen-Name; `render_payload` entfällt. Der Antwort-Text ist der von `departures`; die Rohdaten daneben tragen nur die Felder, die die Ansicht zeichnet. **Anleitung**: `get_usage_guide` mit `tool='departures'`. **Anti-Fab**: Linien, Ziele, Zeiten und Gleise ausschließlich aus dem Output dieses Aufrufs.

| Argument | Required | Type | Description |
|---|---|---|---|
| `arrival_departure` | no | string | `"arrivals"` (when vehicles reach `id`) or `"departures"` (default) — a third value is an argument error. Wins over `is_arrival_time` when both are set. |
| `bearing_tolerance` | no | number | Angular tolerance in degrees for `target_bearing`. Default ±60°. |
| `direction` | no | string | Name of a stop the user wants to head TOWARD („Richtung Wettbergen") — not a routing destination. The board keeps only departures heading that way, whether the named stop is the terminus or a stop downstream of the queried one. Matching is case-insensitive and area-prefix-tolerant. |
| `extended` | no | boolean | `true` keeps the map data in the text: every stop's coordinate, the whole course of each run, and a polyline where the board carries one — for an application that DRAWS the board. It multiplies the answer. A chat client leaves it off. |
| `format` | no | string | Answer serialisation: `"toon"` (default) or `"json"` — the detail is in `get_usage_guide`. **An agent leaves this out.** |
| `id` | yes | string | Stop id to read the board of, as a DHID (`"de:03241:57"`). Resolve the stop first — a name here is an argument error. |
| `is_arrival_time` | no | boolean | `true` reads `time` as an ARRIVAL time — the same as `arrival_departure = "arrivals"`. |
| `last_of_window` | no | boolean | Which end of a `time_range_seconds` window to read: omitted or `false` the FIRST departures, `true` the LAST ones (the end-of-service question). An answer carrying `window_truncated: true` beside its board did NOT reach the window's end — its last entry is then not the last departure. Without a window it has no effect. |
| `limit` | no | integer | Maximum number of departures, at least 1. Default 5, values above 15 are served as 15. |
| `modes` | no | array | Transport-mode filter; the board then shows nothing else. Entries: `"bus"`, `"tram"`, `"subway"`, `"train"`/`"rail"`, `"ferry"`, `"transit"`; several combine. Another entry — a street mode, a misspelling — is an argument error, not a dropped filter: a board lists scheduled services. Omitted or empty = all modes. A Stadtbahn or light-rail mention needs BOTH labels — `["tram","subway"]`: which of the two such a network carries depends on the dataset rather than on the vehicle. Set the filter only when the user really asks for a vehicle type; the guide has the rest of the vocabulary and the traps. |
| `target_bearing` | no | number | Target bearing in degrees (0=N, 90=E, 180=S, 270=W) for a SOFT direction („Richtung Norden", „Richtung Stadt") instead of a stop name. The board keeps only departures whose course lies within `bearing_tolerance` of it. Mutually exclusive with `direction`. |
| `time` | no | string | ISO 8601 timestamp (e.g. `"2026-06-15T08:30:00Z"`). Omit for now. Resolve a spoken time with the time tool rather than computing one. |
| `time_range_seconds` | no | integer | Time WINDOW in seconds, starting at `time`. Together with `last_of_window` this answers „letzte Fahrt": pick it so the window ENDS after end of service (from the evening on, `28800` = 8 h rather than 4 h). Omit for a normal next-N lookup. |
| `verbosity` | no | string | How much of each departure comes back: `"compact"` (default — everything an answer is cited from) or `"full"` (adds the onward stops of the run, and with them the per-stop cancellation marker). A third word is an argument error, not the default. |

## Verbindung schlank anzeigen — `show_connections_slim`

*read-only · not destructive · answers from live outside data*

**Dieselbe Verbindungs-Ansicht wie `show_connections`, nur schlank ausgeliefert**: das Kartenwerk lädt die Ansicht als Skript, statt es mitzubringen. **Nimm dies nur, wenn ausdrücklich nach der schlanken Auslieferung gefragt wird** — sonst `show_connections`. Sonst unverändert: plant die Fahrt wie `connections` und liefert sie zusätzlich als Ansicht aus. **Pflicht sind BEIDE Endpunkte**, wie dort: `origin_id` UND `destination_id` — oder die vier Koordinaten-Felder; `render_payload` entfällt. Der Antwort-Text ist der von `connections`; die Rohdaten daneben tragen nur die Felder, die die Ansicht zeichnet, ihr Kartenverlauf davon im `_meta`. **Anleitung**: `get_usage_guide` mit `tool='connections'`. **Anti-Fab**: Linien, Zeiten, Halte und Gleise ausschließlich aus dem Output dieses Aufrufs.

| Argument | Required | Type | Description |
|---|---|---|---|
| `accessibility_profile` | no | string | Mobility profile by name: `"Standard"` (default), `"WheelchairRobust"`, `"WalkerComfort"` or `"IndividualComfort"`. Each bundles its accessibility constraints and walk speed; another name is an argument error. The two wheelchair profiles find no ride at all today — the answer then carries `accessibility_routing` with `status: no_data` beside ordinary connections, which are to be reported as ordinary ones. What each profile constrains is in the usage guide. |
| `cycling_profile` | no | string | Wer auf dem Rad sitzt, und damit das Tempo der Rad-Abschnitte: `family` (mit kleinen Kindern), `normal` (Default, unverändert) oder `ebike` (Pedelec). Ein anderer Name ist ein Argument-Fehler. |
| `cycling_speed` | no | number | Rad-Tempo in Metern/Sekunde, 1 bis 12 — außerhalb ein Argument-Fehler. Überschreibt `cycling_profile`. Richtwerte: Familie ≈2.8, normal ≈5, Pedelec ≈6.5. |
| `destination_id` | no | string | Destination id. Same form as `origin_id`. |
| `destination_lat` | no | number | Destination latitude. Same use as `origin_lat`. |
| `destination_lon` | no | number | Destination longitude. Goes with `destination_lat`. |
| `destination_type` | no | string | What `destination_id` is. Same values as `origin_type`. |
| `exclude_lines` | no | array | Line blacklist, same token form as `only_lines`. A journey is dropped if any transit leg uses one of them. |
| `extended` | no | boolean | `true` keeps the map data in the text: every stop's coordinate, every leg's polyline, and the whole stop sequence with them — for an application that DRAWS the route, at two to three times the answer. A chat client leaves it off. |
| `format` | no | string | Answer serialisation: `"toon"` (default) or `"json"` — the detail is in `get_usage_guide`. **An agent leaves this out.** |
| `include_rental_bike` | no | boolean | `true` adds a rental-bike leg (park-and-bike / station rental). Equivalent to naming `"bike"` in `modes` together with a transit mode. |
| `is_arrival_time` | no | boolean | `true` reads `time` as the desired ARRIVAL time instead of the departure time. |
| `limit` | no | integer | Maximum number of journeys, at least 1. Default 5, values above 15 are served as 15. |
| `max_transfers` | no | integer | Upper bound on interchanges: `0` = direct, up to `3`. Omit unless the user constrains transfers: the backend then derives the search depth from the trip's air-line distance and escalates it when the first attempt finds nothing, so omitting never loses a connection. |
| `max_walk_meters` | no | integer | Cap on the summed walking DISTANCE in metres. Works like `max_walk_minutes`, on the metres walked. |
| `max_walk_minutes` | no | integer | Cap on the summed walking TIME in minutes. A post-filter drops journeys above it; if that leaves nothing, the backend searches once more avoiding footpaths before the answer is empty. |
| `mobility_profile` | no | string | How the first and the last mile may be covered, by name: `"transit"` (default — both ends on foot), `"transit_plus_sharing"` (a shared bike), `"door_to_door"` (being driven), `"park_and_ride"` or `"bike_and_ride"` (one's OWN car or bicycle, left where the transit leg starts). Another name is an argument error. It widens those two ends, restricts no ride and promises nothing. When to set which, and how it composes with `modes`, is in the usage guide. |
| `modes` | no | array | Transport-mode restriction, not a preference: nothing outside the list is planned. The footpath always stays allowed — the way to, from and between stops goes on foot — while `["foot"]` on its own plans no ride at all. Entries: `"bike"`, `"bike_rental"`, `"foot"`, `"car"`, `"taxi"`, `"scooter"`, `"transit"`, `"bus"`, `"tram"`, `"rail"`, `"subway"`, `"ferry"`; several combine. An entry outside that list is an argument error, not a dropped filter. Omitted or empty = all standard transit modes, the ferry included. Set only when the user insists on a mode; the guide has the vocabulary and the traps. |
| `only_lines` | no | array | Line whitelist. Entries are line tokens as printed on the vehicle (`"6"`, `"U3"`); a journey is kept only if every transit leg uses one of them. |
| `origin_id` | no | string | Origin id. A DHID (`de:NNNNN:NNN`) for a stop, otherwise the resolver's `main` id of the address or POI. Resolve the place first — a name here is an argument error. |
| `origin_lat` | no | number | Origin latitude, WGS-84 decimal degrees — the fallback when no id is at hand, and the one retry worth making when resolved ids find nothing. |
| `origin_lon` | no | number | Origin longitude. Goes with `origin_lat`. |
| `origin_type` | no | string | What `origin_id` is: `"stop"` (default), `"address"` or `"poi"`. A stop id is auto-prefixed `GTFS.de:`; the other two are forwarded verbatim so the backend resolves the real name. |
| `prefer_flat` | no | boolean | `true` plant die hügel-ärmste statt der sonst gewählten Rad-Route: die Suche gewichtet die Steigung und nimmt dafür Umwege in Kauf, nur auf Rad-Abschnitten. Keine Zusage — ohne flachere Alternative kommt dieselbe Route; die Höhenmeter der Antwort sagen es. Wann setzen: die Anleitung. |
| `submodes_allow` | no | array | Fine submode whitelist — finer than `modes`, which only separates bus/tram/rail/subway. Entries: `"sbahn"`, `"regionalbahn"`, `"ice"`, `"ic"`, `"ir"`, `"nj"`, `"fernverkehr"`, `"stadtbahn"`, `"ubahn"`, `"bus"`, `"regionalbus"`, `"stadtbus"`. A journey is kept only if every transit leg matches one. |
| `submodes_deny` | no | array | Fine submode blacklist, same vocabulary as `submodes_allow`. A journey is dropped if any transit leg matches one. |
| `time` | no | string | ISO 8601 timestamp (e.g. `"2026-06-15T08:30:00Z"`). Omit to plan from now. Resolve a spoken time with the time tool rather than computing one. |
| `verbosity` | no | string | How much of each journey comes back: `"compact"` (default — everything an answer is cited from) or `"full"` (adds the stops in between). A third word is an argument error, not the default. |
| `via_id` | no | string | Intermediate stop the route must pass through („über X"), as an id in the same form as `origin_id`. The route is then planned A → via → B. |
| `via_lat` | no | number | Via latitude, for an intermediate address or POI without an id. Goes with `via_lon`. |
| `via_lon` | no | number | Via longitude. Goes with `via_lat`. |
| `via_type` | no | string | What `via_id` is. Same values as `origin_type`. |
| `walk_speed` | no | number | Walking speed of the foot legs in metres/second, between 0.3 and 3 m/s — outside that it is an argument error. Default ≈1.33, slow ≈0.7, brisk ≈2.0. Overrides the walk speed of `accessibility_profile`. |

<!-- tools:end -->
