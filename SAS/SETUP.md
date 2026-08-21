# Trade Lane Watch — setup

`trade-lane-watch.html` is the whole object: markup, styling, animation and
coastline geometry in one file. `ship_movements_bl.csv` is the sample data to
load into CAS.

## Where the map gets its data

The object never reads a file. There are exactly two sources:

1. **Built-in preview** — 288 sample rows compiled into the file, so the page
   works standalone before VA is wired up.
2. **SAS Visual Analytics** — VA queries CAS and posts the result into the
   iframe. The first message replaces the preview completely.

A badge in the top bar tells you which one you are looking at:

- amber `built-in preview data · 288 rows` — not connected
- blue `CAS · 288 rows · 9 vessels · 14:32` — live, with the time of the last
  result received

Add rows in CAS, refresh the report, and the badge count, the KPIs, the fleet
legend and the timeline all change with it. To remove any chance of confusion
in production, delete the `window.DEMO_DATA = {...}` statement from the script
block; the object then shows "Waiting for data" until VA connects.

## 1. Load the CSV into CAS

```sas
cas mysess;

proc casutil;
   load casdata="ship_movements_bl.csv" incaslib="Public"
        casout="SHIP_MOVEMENTS" outcaslib="Public"
        importoptions=(filetype="csv" getnames=true guessrows="max")
        promote;
   list tables incaslib="Public";
quit;
```

`promote` is what makes the table visible to VA. Without it the table is
session-scoped and the report will not find it.

Then confirm the typing — `latitude`, `longitude`, `t_hours`, `speed_knots`,
`heading` and `ping_seq` must arrive **numeric**. If `latitude` lands as
character the object cannot project it and shows its empty state.

## 2. Point the object at the file

Objects -> Data-Driven Content, set *URL* to wherever the HTML is hosted, over
https. Then Options -> **Maximum rows** -> above your ping count (288 for the
sample). VA's default truncates silently and the tracks come out chopped.

## 3. Assign roles

**Required**

| Column | Role | Aggregation |
|---|---|---|
| `vessel_aircraft` | category | — |
| `latitude` | measure | **Average** |
| `longitude` | measure | **Average** |
| `t_hours` | measure | **Average** |

**Categories** — `voyage_no`, `ping_seq`, `ping_time`, `shipper`,
`shippers_reference`, `place_of_receipt`, `port_of_loading`,
`port_of_discharge`, `place_of_delivery`, `final_destination`

**Measures, Average** — `speed_knots`, `heading`

Average is not cosmetic. VA aggregates by the category roles, so with
`vessel_aircraft` + `voyage_no` + `ping_seq` as categories each group holds one
ping and the average of a position is that position. Sum collapses every vessel
to one point in the wrong ocean.

## Adding your own data

One row per position report. The only hard requirements are a stable vessel
key, latitude, longitude and a numeric hour offset. Document fields repeat on
every ping — denormalised on purpose, since a DDC object receives one result
set. To build `t_hours` from a datetime:

```sas
t_hours = intck('SECOND', min_ping_dttm, ping_dttm) / 3600;
```

New vessels appear automatically, each with its own colour and legend chip. The
map only re-frames itself when the fleet roster changes, so a filter refresh
will not pull the view away from wherever you were looking.

## Content-Security-Policy

Serving this from SAS Content fails: Viya applies `default-src 'self'`, which
blocks inline script and style. Options, best first:

1. **Host it on an ordinary web server** — internal IIS, nginx, Apache. Static
   hosts normally send no CSP and the file runs as built.
2. **Ask an admin to allow these two hashes** on the serving host. They are in
   the comment at the top of the file and authorise this one artifact rather
   than blanket `unsafe-inline`. Note they change whenever the file is edited.
3. **Use the split build** (`ship-tbml-map.html` + `.css` + `.js` in the same
   folder), which needs no inline execution — but needs real folder paths, so
   SAS Content is still out.

## Logo

The header has a slot marked `ADIB LOGO SLOT` with a neutral placeholder in it.
Paste the official mark from the brand team as **inline SVG** — under
`default-src 'self'` an `<img src>` needs `img-src 'self'` and a data URI needs
`img-src data:`, while inline SVG is markup and needs neither. The CSS sizes it
to 30px tall.
