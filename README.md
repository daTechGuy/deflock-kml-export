# deflock-kml-export

Pull ALPR (automated license plate reader) camera locations straight from
OpenStreetMap -- the same live data [DeFlock.me](https://deflock.me) itself
displays -- and write them out as KML files you can drop into Google My Maps,
Google Earth, or QGIS.

DeFlock doesn't run its own database. Every camera on its map comes from
OpenStreetMap nodes tagged `surveillance:type=ALPR`, queried live via the
[Overpass API](https://overpass-api.de/). This script runs that same query
directly, so what you get is exactly as current as DeFlock's own map, not a
stale export.

No third-party dependencies -- standard library only. Anyone with Python 3
installed can run this with no `pip install` step.

## Usage

```
python deflock_kml.py --state California
python deflock_kml.py --state CA
python deflock_kml.py --all-us
python deflock_kml.py --list-states
```

Options:

| Flag | Description |
|---|---|
| `--state NAME` | One US state, by full name or 2-letter code (e.g. `California` or `CA`) |
| `--all-us` | All 50 states + DC, fetched one state at a time |
| `--list-states` | Print valid `--state` values and exit |
| `--output-dir DIR` | Where to write the KML file(s) (default: current directory) |
| `--max-per-file N` | Max placemarks per KML file (default: 2000 -- see below) |

Each run writes a KML file named like `deflock_ca_2026-08-22.kml` into the
output directory.

See [`examples/`](examples/) for real output from `--state` runs, one
subfolder per state (Alabama, Alaska, Arizona, Arkansas, California, Colorado,
Connecticut, Delaware, Florida, Georgia, Hawaii, Idaho, Illinois, Indiana,
Iowa, Kansas, Kentucky, Louisiana, Maine, Maryland, Massachusetts, Michigan,
Minnesota, Mississippi, Missouri, Montana, Nebraska, Nevada, New Hampshire,
New Jersey, New Mexico, New York, North Carolina, North Dakota, Ohio,
Oklahoma, Oregon, Pennsylvania) -- including the automatic split into
multiple files by the 2,000-per-layer rule described below (California alone
splits into 11 files at 20,197 nodes).

## Why per-state, not one national query

`--all-us` fetches each state individually rather than issuing one massive
national query. During development, pulling the eastern and western halves
of the US in two big requests returned 80,000+ nodes each and got the
requesting IP rate-limited (HTTP 429) and then temporarily connection-blocked
by the public Overpass mirror entirely. Per-state queries are small enough to
stay well inside normal usage, and if one state's query fails, only that
state needs a retry -- not the whole country.

`--all-us` still takes a while (51 sequential queries, ~1 second of
deliberate throttling between each, plus real query time) -- expect it to run
for several minutes.

## Importing into Google My Maps

Google My Maps has hard limits that this tool works around automatically:

- **2,000 features per layer.** Exceeding this doesn't error -- Google
  silently drops everything past the limit. `deflock_kml.py` splits output
  into multiple `_partXofY.kml` files at this boundary by default, so each
  file imports cleanly as one layer.
- **10 layers / 10,000 features per map, total.** If a state (or `--all-us`)
  produces more than that, you'll need more than one My Maps map, or use
  Google Earth Pro / QGIS instead, which don't have this ceiling.

To import: My Maps -> Create a new map -> Import -> select a `.kml` file.
Each file becomes one layer.

## Data source and license

Camera data comes from [OpenStreetMap](https://www.openstreetmap.org/), and
is licensed under the [Open Database License (ODbL)](https://www.openstreetmap.org/copyright).
If you republish or redistribute the data (not just use it privately), you're
bound by ODbL's share-alike and attribution terms.

The public Overpass API mirrors this script queries are free, shared
infrastructure. Please don't run `--all-us` in a tight loop -- the built-in
throttling is there deliberately, not just for show.
