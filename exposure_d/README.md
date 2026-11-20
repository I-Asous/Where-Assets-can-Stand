# NYC Wind Exposure D × MTA Bus Stops

Maps where NYC Building Code **Exposure D** (high-wind coastal zone) applies, and which MTA bus stops fall inside it. Built for NYC DOT, to flag bus poles that may need stronger foundations before new devices are mounted on them.

## Contents

| Path | What it is |
|---|---|
| `data/exposure_d/exposure_d_final_v2_shp.zip` | **Exposure D zones**: zipped shapefile, EPSG:2263, 66 polygons ("D bodies"). The same layer is also unzipped alongside. |
| `data/exposure_d/exposure_d_final_v2.geojson` | The same zones in EPSG:4326, for web use. |
| `data/bus_stops/stops_d_overlay_shp.zip` | **Bus stops in or near D**: zipped shapefile, EPSG:2263, 485 stops. Also as `.csv` and `.geojson` (EPSG:4326). |
| `data/counts/d_counts_by_body.csv` | Stop counts for each D body, with the routes that serve it. |
| `data/counts/d_counts_by_borough.csv` | Stop counts for each borough. |
| `data/working_layers/` | Source layers: `d_shoreline` (traced D shorelines) and `exposure_d` (whole-island D polygons). |

## Fields

**exposure_d_final_v2**: `d_id` (1–66), `borocode`, `boroname`, `acres`, `basis`

**stops_d_overlay**
- `stop_id`, `stop_name`, `stop_lat`, `stop_lon`
- `routes`: routes that serve the stop
- `status`:
  - `in`: inside D, more than 300 ft from the D/C line
  - `edge`: inside D, within 300 ft of the D/C line
  - `near`: outside D, within 300 ft of the D/C line
- `d_id`, `boroname`
- `dist_to_d_edge_ft`: distance to the inland D/C line. Blank for whole-island D.

## Method

1. **Source figures.** Figures 1609.4.3(1)–(5) in the 2022 NYC Building Code, Chapter 16. Each figure was georeferenced in QGIS (Polynomial 1). The double-line ("D coastal zone") shorelines were then traced onto the NYC Open Data borough boundaries.
2. **The D band.** §1609.4.3 puts D within 600 ft, or 20 × building height if that is greater, inland of a D shoreline. Poles are under 30 ft, so 600 ft governs.
3. **Building the zones.** The traced shorelines were buffered by 600 ft and clipped to land. Islands that the figures mark as entirely D were then merged in: Governors, Ellis and Liberty; City, Hart and the other Bronx islands; and every Jamaica Bay island. The result was fixed, dissolved, split into separate bodies, and tagged by the borough with the largest overlap.
4. **Bus stops.** Taken from the MTA NYCT and MTA Bus GTFS feeds (13,465 stops, valid Oct 12, 2026 – Jan 2, 2027). Each stop was tested against the D zones by point-in-polygon. Distance was measured to the inland D/C boundary only, ignoring the waterline.

## Results

| Borough | D bodies | Bodies with stops | Stops in D | ≤300 ft from line | Outside, ≤300 ft |
|---|---|---|---|---|---|
| Bronx | 21 | 2 | 50 | 17 | 22 |
| Brooklyn | 19 | 2 | 128 | 58 | 83 |
| Manhattan | 3 | 0 | 0 | 0 | 0 |
| Queens | 22 | 3 | 29 | 15 | 41 |
| Staten Island | 1 | 1 | 47 | 36 | 85 |
| **Total** | **66** | **8** | **254** | **126** | **231** |

## Assumptions and limitations

- **Stop locations.** GTFS stop points are used as a stand-in for pole locations.
- **Georeferencing error.** It is up to a few hundred feet in places. Treat `edge` and `near` stops as borderline and check them in the field.
- **Whole-island D.** City Island (Bx29) and Broad Channel (Q52/Q53) count as D across the whole island, including points more than 600 ft from shore, because the figures mark them that way.
- **Body 14.** It crosses the Brooklyn/Queens line at Spring Creek and is labeled Brooklyn, so its Queens-side stops (Q routes) count toward Brooklyn.
- **Not an official determination.** This is not an official DOB exposure map. Confirm against the code text for any design decision.

## Sources

- NYC Building Code §1609.4.3 and 2022 BC Chapter 16, Figures 1609.4.3(1)–(5)
- NYC Open Data: Borough Boundaries (water areas excluded)
- MTA GTFS static feeds: `gtfs_b`, `gtfs_bx`, `gtfs_m`, `gtfs_q`, `gtfs_si`, `gtfs_busco`
