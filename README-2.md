# EduCity F3 PostGIS Pack

**EduCity, third floor** · Joukahaisenkatu 7, Turku, Finland  
Release `educity-f3-20261105-postgis-v2` · Site key `EDUCITY-TURKU`

This revision is synchronized to the supplied 2026-11-05 preview. It updates sign positions and associations, removes ES-09, applies the WC03 double-swing egress geometry, and refreshes the current door-access records. The traced floorplate, zones, and stair-detail layers are retained.

This developer handoff follows the validated 7 City Road F7 package’s table, CSV, GeoJSON, workbook, and SQL layout. It contains the color-up floorplate and the 16 exit sign positions in the supplied preview. The source-image coordinates are kept in the registers for traceability.

## Project and documentation scope

This project demonstrates AI-assisted interpretation and tracing of an architectural floor-plan image into an attributed indoor GIS dataset. Paul Chan supplied the source guidance and corrected exit-sign markup, specified the PostGIS format, and reviewed the results. ChatGPT/Codex interpreted the drawing, recorded trace coordinates, generated the processing code, and produced the GIS exports and validation reports.

The included scripts rebuild the dataset from recorded trace picks and approved inputs. The supplied source is an image of the architectural plan. Processing an original PDF page, selecting its scale and crop, and recording those preparation steps form the input stage when starting from a PDF.

**Dataset release** `educity-f3-20261105-postgis-v2`  
**Documentation consolidated** 8 November 2026  
**Project** IndoorSOS / 2-3D AI mapping

The relative links and commands below assume this README is placed alongside the extracted package's `source/`, `sql/`, `data/`, `geojson/`, `assets/`, and `qa/` directories in the GitHub repository.

![EduCity F3 geocoded floor plan with zones, door access, exit signs and stair detail](assets/EduCity_F3_Map.png)

## AI-assisted workflow, tools and languages

The sequence follows the 7 City Road F7 package structure. Each building has its own source drawing, geometry, geographic controls, IDs and review decisions.

| Step | Task and recorded method | Tools used | Language and implementation |
|---|---|---|---|
| **1. Establish the source drawing** | Identify the site, floor and source revision. Inspect the supplied plan image and corrected markup. Record crop and drawing coordinates. For a PDF input, identify the page and retain the original page-to-image relationship. | Visual review and Pillow image crops during preparation. | **Python** assisted image inspection. The EduCity trace inputs are retained in [approved_trace.json](source/approved_trace.json) and [assets/](assets/). |
| **2. Define the data structure and IDs** | Reuse F7 table and register conventions. Assign stable site, floor, zone, access and sign IDs. Keep mapped sign IDs separate from physical hardware IDs. | Python `json` and `csv`, the F7 column contract and PostGIS schema. | **Python and SQL** in [f7_column_contract.json](source/f7_column_contract.json), [build_gis.py](source/build_gis.py) and [00_schema.sql](sql/00_schema.sql). |
| **3. Trace the floorplate** | Visually pick and order the plan boundary vertices, including the visible ICT-City link. Build a closed floorplate polygon. | Recorded trace picks and Shapely. | **Python** in [build_gis.py](source/build_gis.py). |
| **4. Build zone polygons** | Trace rooms, circulation, stairs and shafts. Insert shared boundary junctions and export the ordered vertex and boundary registers. | Shapely geometry construction and validation. | **Python** in [build_gis.py](source/build_gis.py). |
| **5. Georeference the drawing** | Match four drawing corners to mapped building corners. Fit an affine transformation in the local metric CRS and record the controls and residuals. | OpenStreetMap footprint snapshot, NumPy least-squares fitting and pyproj. | **Python** in [build_gis.py](source/build_gis.py), with controls in [georeference.json](data/georeference.json). |
| **6. Generate geographic and metric geometry** | Apply the same transformation to each feature. Store WGS 84 longitude and latitude for geographic exports and metric geometry for the GeoPackage and measurements. | pyproj/PROJ, Shapely and Pyogrio/GDAL. | **Python** in [build_gis.py](source/build_gis.py). |
| **7. Trace doors and openings** | Pick jambs and visible hinges. Generate thresholds and leaf swings and associate them with rooms. Apply the approved V2 access correction. | Shapely LineString construction and the recorded door picks. | **Python** in [room_door_trace.py](source/room_door_trace.py), [room_doors.csv](source/room_doors.csv) and [door_access_final_overrides.json](source/door_access_final_overrides.json). |
| **8. Place signs and link egress records** | Transfer the corrected sign positions, associate them with access records, and preserve the approved preview orientations and supplied hardware IDs. | Python data processing, ImageGen for the supplied fixture-photo cutout during preparation, and SVG transforms for display. | **Python and SVG** in [build_gis.py](source/build_gis.py) and [build_sign_display.py](source/build_sign_display.py), with the artwork in [exit-sign-fixture.png](assets/exit-sign-fixture.png). |
| **9. Trace stair detail** | Record visible tread-line positions. Construct flight bands, tread polygons and interpreted landings. Keep terrace candidates marked for review. | Visual inspection of enlarged source crops and Shapely polygons. | **Python** in [stair_flight_trace.py](source/stair_flight_trace.py) and [stair_flight_trace.csv](source/stair_flight_trace.csv). |
| **10. Export the deliverables** | Write matching CSV, GeoJSON, GeoPackage, KML and SQL outputs and the Excel register. | Python `csv`, `json` and `ElementTree`, Pyogrio/GDAL, and `@oai/artifact-tool` for Excel. | **Python and JavaScript on Node.js** in [build_gis.py](source/build_gis.py), [workbook_input.py](source/workbook_input.py) and [build_register.mjs](source/build_register.mjs). |
| **11. Load PostGIS** | Create the schema, load typed geometry, create views and run the checks within the standalone transaction. | PostgreSQL/PostGIS, `psql` for the documented database load, and PGlite/PostGIS for the recorded local test. | **SQL**, assembled by **Python** in [build_standalone.py](source/build_standalone.py). The load order is `00_schema.sql`, `01_load.sql`, `03_views.sql`, then `02_validate.sql`. |
| **12. Validate database and exports** | Check counts, IDs, geometry validity, containment, overlaps, sign/access links and stair relationships. Compare exports and repeat the database load. | PostGIS functions, PGlite, Shapely, Pyogrio and SQLite. openpyxl was used for workbook readback during preparation. | **SQL, JavaScript and Python** in [02_validate.sql](sql/02_validate.sql), [verify_postgis.mjs](source/verify_postgis.mjs) and [verify_exports.py](source/verify_exports.py). |
| **13. Review and release** | Inspect the geographic preview, compass, signs, doors and stair insets. Package the source and outputs and record file checksums. | Browser display, Sharp for the PNG render during preparation, and Python `zipfile` and `hashlib` for packaging and checksums. | **Python, HTML, CSS, SVG and JavaScript** in [build_preview.py](source/build_preview.py). The published file list is [SHA256SUMS.txt](SHA256SUMS.txt). |

The visual interpretation and trace picks were reviewed through the conversation. The build scripts apply those recorded inputs consistently. ImageGen prepared the fixture artwork. The spatial geometry is constructed by the GIS code from the trace inputs.

### Software used to rebuild the package

| Component | Role |
|---|---|
| Python 3.11+ | Geometry processing and exports. Install the Python packages from [source/requirements.txt](source/requirements.txt). |
| NumPy, Shapely, pyproj and Pyogrio | Affine fit, geometry construction, CRS transformation and vector-file output. |
| PostgreSQL with PostGIS and the `psql` client | Database load and validation. The recorded embedded test used PostgreSQL 18 and PostGIS 3.6.2 through PGlite. |
| Node.js and npm | Run the JavaScript database verifier and workbook builder. |
| `@electric-sql/pglite` and `@electric-sql/pglite-postgis` | Repeat the embedded PostGIS verification using the pinned versions below. |
| `@oai/artifact-tool` | Regenerate the Excel register in an environment where this package is available. |
| A web browser | Open the included HTML preview and inspect the layers and detail views. |
| QGIS | An optional client for viewing the PostGIS tables or GeoPackage. |

Pillow, ImageGen, Sharp and openpyxl were used for source inspection, asset preparation or readback during development. The core GIS rebuild uses the dependencies listed in `source/requirements.txt`.

## Load and display

Extract the full ZIP. Run from the extracted pack directory with a PostgreSQL database connection. The database must have PostGIS installed or allow the loader role to install the extension.

```bash
psql -X "$DATABASE_URL" -v ON_ERROR_STOP=1 -f sql/99_standalone_full_load.sql
```

The standalone loader creates the `educity_f3` schema, loads this release, creates the views, and checks the data in one transaction. Re-running it refreshes this release. For staged loading, use `00_schema.sql`, `01_load.sql`, `03_views.sql`, and `02_validate.sql` in sequence.


For a staged load, run each command from the extracted package directory.

```bash
psql -X "$DATABASE_URL" -v ON_ERROR_STOP=1 -f sql/00_schema.sql
psql -X "$DATABASE_URL" -v ON_ERROR_STOP=1 -f sql/01_load.sql
psql -X "$DATABASE_URL" -v ON_ERROR_STOP=1 -f sql/03_views.sql
psql -X "$DATABASE_URL" -v ON_ERROR_STOP=1 -f sql/02_validate.sql
```

In a PostGIS client such as QGIS, add `educity_f3.floorplate`, `educity_f3.zones`, and `educity_f3.exit_signs`. Use `zone_id` for zone labels and `display_color` for fills. Use the 4326 `geom` column. Filter by release ID if the schema contains multiple releases. Add `v_egress_signs` when you want the adjacency register joined to the sign points. Its geometry is the sign location. Add `educity_f3.v_room_door_access` to display all threshold and swing linework. Filter on `part = 'threshold'` for one record per access, or on `zone_id` and `adjacent_zone_id` for a room. The D1-only view remains available as `educity_f3.v_entrance_egress`. Add `educity_f3.v_stair_flights` for the 14 flight bands or `educity_f3.stair_flight_detail` for the complete stair detail. The equivalent files for offline viewing are `EduCity_F3.gpkg` and `EduCity_F3_PostGIS_Preview.html`.

```sql
SELECT zone_id, space, display_color, area_m2, geom
FROM educity_f3.v_zone_summary
WHERE release_id = 'educity-f3-20261105-postgis-v2';

SELECT check_name, expected, observed, pass
FROM educity_f3.v_release_qa
WHERE release_id = 'educity-f3-20261105-postgis-v2'
ORDER BY check_name;
```

## Contents

| Layer or register | Features | PostGIS representation |
|---|---:|---|
| Floorplate | 1 | Polygon |
| Floorplate vertices | 30 | Point |
| Zones | 52 | Polygon |
| Zone vertices | 340 | Point |
| Zone wall segments | 340 | LineString |
| Stair cores | 4 | Polygon |
| Visible stair runs | 14 | Polygon |
| Visible stair treads | 83 | Polygon |
| Interpreted landing areas | 14 | Polygon |
| Terrace review candidates | 2 | Polygon |
| Lift shafts | 4 | Polygon in `zones` and `v_lift_shafts` |
| Corrected exit signs | 16 | Point |
| Egress adjacency | 16 | Register joined to sign points by `v_egress_signs` |
| Room door and opening access | 66 thresholds, 70 swing lines | LineString parts in `door_access` |
| Lift cars and single doors | 0 | Empty compatibility tables |

The 340 zone vertices comprise 275 traced vertices and 65 existing junctions inserted along shared edges before reprojection. The boundary segment register includes review subdivisions. Use it as zone linework. Door linework follows the source drawing and the existing georeference. Each access has a threshold and zero, one, or two leaf swing lines. The 66 locations comprise 40 single-leaf doors, 15 double-leaf doors, seven opening-only traces requiring review, and four lift entrance spans. All 16 current sign positions are linked to their corresponding access records. These associations describe plan locations. Travel directions and field conditions remain unverified.

`data/room_door_coverage.csv` gives one disposition per frozen zone. Six polygons have no distinct doorway traced. N9 shares its drawn enclosure with N8, whose north door is indexed under N8. C1–C4 and RF1 have no readable discrete door symbol. Where a wall gap or parallel strokes are visible but the leaf is uncertain, only the opening is traced. Lift entries are traced as spans without lift-car geometry. Door widths are approximate drawing-derived values. The `image_left`, `image_right`, `image_up`, and `image_down` swing fields refer to the architectural drawing axes. Source jamb picks and swing directions are in `source/room_doors.csv`.

| File or directory | Use |
|---|---|
| `sql/` | Schema, typed geographic load, checks, display views, and standalone loader |
| `data/` | F7-compatible CSV registers, georeference controls, and canonical snapshot |
| `geojson/` | Individual WGS84 layers and one combined FeatureCollection |
| `EduCity_F3.gpkg` | Ten populated layers in the local metric CRS |
| `EduCity_F3.kml` | Floorplate, zones, sign points, stair cores, stair detail, and door linework for globe viewers |
| `EduCity_F3_Vertices_and_Features.xlsx` | F7 register workbook with `FW Adv id` columns and door coverage, stair flight, and stair vertex registers |
| `EduCity_F3_PostGIS_Preview.html` | Offline GIS preview with zone fills, sign markers, selectable door, sign, and stair detail insets, compass, and scale |
| `assets/` | Source floor plan, corrected sign reference, map render, and earlier satellite review |
| `source/`, `qa/` | Build inputs, runnable code, and recorded validation |

## Stair flight detail

| Core | Visible flight bands | Visible tread polygons | Interpreted landings |
|---|---:|---:|---:|
| S01 | 3 | 21 | 3 |
| S02 | 3 | 21 | 3 |
| S03 | 4 | 21 | 4 |
| S04 | 4 | 20 | 4 |

The tread polygons are the spaces between consecutive tread lines visible in the architectural drawing. A flight band covers its visible tread polygons, so those two layers intentionally overlap. Some bands are cut short in the drawing. The `tread_index` numbers follow increasing source-image coordinates on the run axis. Positions such as `PLAN-T` are sheet positions. Landing footprints are interpreted from the clear areas adjacent to the runs. Two plausible R1 terrace features are supplied as draft review outlines, without tread counts. The long striped edge along the larger terrace platform is visually ambiguous and was excluded.

`geojson/stair_flight_detail.geojson` and the PostGIS polygon layer use WGS 84 longitude and latitude. `data/stair_flight_vertices.csv` and the workbook's **Stair Vertices** sheet provide both WGS 84 and EPSG:3877 coordinates for all polygon corners. `source/stair_flight_trace.csv` preserves the measured drawing picks. This is 2D drawing-derived geometry. Floor connections, UP/DOWN directions, elevations and the full built tread counts remain unverified.

## Exit-sign display alignment

The preview fixture artwork is in `assets/exit-sign-fixture.png`. The geographic preview and satellite review each embed the artwork and apply per-sign rotations. Select a sign on the geographic preview to open its linked doorway detail. The display register in `data/exit_sign_display.csv` lists each sign and access ID, the SVG and drawing rotation, and the true-north axes along and across the doorway. Those axes are undirected. The icon rotation describes fixture alignment for the map. The travel direction and installed fixture facing are unverified.

The sign display register follows the supplied preview. The four assigned FW Adv IDs are HLP006400990000011 (ES-06), HLP006500990000011 (ES-07), HLP006600990000011 (ES-10), and HLP006700990000011 (ES-11). Display orientation follows the approved preview; field travel direction remains unverified.

## Registration and field status

PostGIS geometry and GeoJSON use WGS 84, EPSG:4326, in longitude and latitude order. The GeoPackage and measurements use ETRS89 / GK23FIN, EPSG:3877, in metres. The compass shows true bearings. The plan is two-dimensional. Floor elevation is blank.

The four main rectangular corners of the floor drawing were matched to the corresponding corners of OpenStreetMap way 731925812, version 19, and fitted by affine transformation in EPSG:3877. The exact corner pairs, fit coefficients, and residuals are in `data/georeference.json`. The small residual measures fit to those mapped controls. Independent survey accuracy has not been established. The release records `survey_grade=false`. The traced floorplate includes the portion of the ICT-City link visible on the plan. The full OSM outline is a separate control reference. The earlier satellite review retains the visual roof-cover overlay.

Sign points preserve the corrected marked locations and the local `EDU-F3-ES-xx` IDs. Four `FW Adv id` values are assigned as listed in the exit-sign register. Sign rotation follows the supplied preview. Egress records describe sign adjacency. Their `from_area` and `to_area` columns preserve the F7 layout. Travel direction, route connectivity, and floor-to-floor pairings are unverified. ES-01 and ES-14 have an unknown second adjacent area and a null `to_area`.

Footprint source © OpenStreetMap contributors. [Mapped footprint](https://www.openstreetmap.org/way/731925812) · [Data terms](https://www.openstreetmap.org/copyright)

## Rebuild and verify

Use a working copy of the extracted package when rebuilding. Check the delivered checksums before regenerating files.


Run the GIS builder with Python 3.11+ and the packages in `source/requirements.txt`. The source trace and footprint snapshot are bundled. Rebuilding the core GIS files requires no live map service.

```bash
python3 -m venv .venv
. .venv/bin/activate
python -m pip install -r source/requirements.txt
python source/build_gis.py
python source/build_standalone.py
python source/build_sign_display.py
python source/build_preview.py
python source/verify_exports.py
```

Use `sha256sum -c SHA256SUMS.txt` to check the delivered files **before** rebuilding. Generated container files can include fresh metadata on rebuild.

The included PostGIS loader and validation SQL are updated for this release. The preview-synchronized release was validated with PostgreSQL 18 and PostGIS 3.6.2 through PGlite: 32 SQL checks passed, the standalone loader succeeded twice, and sign coordinates matched GeoJSON. The 2026-11-04 validation report is retained separately as a historical record. Frozen GeoJSON layer fingerprints are recorded in `source/locked_geojson_sha256.json`. The export registers, GeoPackage, KML, and workbook carry the same current feature counts.

```bash
npm install --no-save @electric-sql/pglite@0.5.15 @electric-sql/pglite-postgis@0.2.15 sharp
node source/verify_postgis.mjs
```

The workbook is editable and included. To regenerate it in an environment with `@oai/artifact-tool` installed, run the following after PostGIS verification.

```bash
python source/workbook_input.py > /tmp/educity-workbook.json
node source/build_register.mjs /tmp/educity-workbook.json EduCity_F3_Vertices_and_Features.xlsx
```


### Rebuild inputs and preview behaviour

V2's [approved_trace.json](source/approved_trace.json) contains the current sign records. [door_access_final_overrides.json](source/door_access_final_overrides.json) supplies the approved access correction. Retain both inputs when rebuilding V2.

The preview builder preserves the supplied [final_preview.html](source/final_preview.html) when that file is present. It also extracts the corresponding inline SVG into `assets/EduCity_F3_Map.svg`. Edit the approved preview and source inputs together when making a future reviewed revision.

[build_preview.py](source/build_preview.py) writes HTML and SVG. The PNG render is a separate step. The npm command in the verification section installs Sharp alongside the verifier dependencies. The PNG can then be regenerated from the SVG.

```bash
node --input-type=module -e 'import sharp from "sharp"; await sharp("assets/EduCity_F3_Map.svg", { density: 144 }).png().toFile("assets/EduCity_F3_Map.png");'
```

The recorded V2 validation results are in [qa/postgis_validation.json](qa/postgis_validation.json) and [qa/export_validation.json](qa/export_validation.json). Those files document the release test. Run the verification commands above to create a new test result in your own environment.

## Historical interaction and timing record

The paired slots below document selected V1 development exchanges on **3 and 5 November 2026**, in **Pacific/Auckland time (NZDT, UTC+13)**. They cover the initial trace, mapping revisions and developer handoff. V2 incorporates subsequent preview and hardware-ID revisions. A separate timing measurement for those V2 revisions is unavailable.

A **prompt-to-response slot** starts when Paul sends a request and ends when the assistant delivers the corresponding response. For example, the PostGIS rebuild ran from 04:10:32 to 04:26:28, an elapsed slot of **15 minutes 56 seconds**. Each slot includes any waiting within that exchange. Gaps between exchanges and steps without paired timestamps are excluded from the totals. Measurements of active AI processing and human work are unavailable.

| Date, NZDT | Recorded exchange | Prompt | Response | Elapsed, min:sec |
|---|---|---|---|---|
| 3 Nov | First floor trace | 00:34:24 | 00:46:25 | 12:01 |
| 3 Nov | Referee-suggested zones | 00:49:52 | 00:53:34 | 3:42 |
| 3 Nov | Corrected exit signs and roof overlay | 01:41:41 | 01:47:27 | 5:46 |
| 3 Nov | Colour roof overlay | 01:51:15 | 01:58:36 | 7:21 |
| 5 Nov | Developer handoff | 00:09:24 | 00:16:33 | 7:09 |
| 5 Nov | F7-format PostGIS rebuild | 04:10:32 | 04:26:28 | 15:56 |
| 5 Nov | D1 entrance egress | 04:33:51 | 04:38:57 | 5:06 |
| 5 Nov | Remaining room doors | 04:41:41 | 04:50:41 | 9:00 |
| 5 Nov | Exit-sign icon replacement and alignment | 04:55:43 | 05:01:31 | 5:48 |
| 5 Nov | Geocoded stair detail | 05:05:07 | 05:15:02 | 9:55 |
| 5 Nov | Workflow write-up | 05:19:02 | 05:19:49 | 0:47 |
| 5 Nov | Tools and languages write-up | 05:22:44 | 05:23:05 | 0:21 |

**Recorded mapping and handoff slots total 1 hour 21 minutes 44 seconds.** Including the workflow and tools write-ups, the recorded total is **1 hour 22 minutes 52 seconds**.

This is a reconstructed summary of selected interactions. The source-to-output evidence begins with the architectural plan image and recorded picks. The package supplies the code, geographic controls, exports and validation reports for review. A recorded run beginning with the original PDF and an independently timed manual-work comparison would provide additional evidence for assessing the mapping process.

## Updating this README

This consolidated README is intended to replace the package README in the repository. After a reviewed documentation change, refresh the `README.md` entry in `SHA256SUMS.txt` so the checksum list matches the uploaded file.

