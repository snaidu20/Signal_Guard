# SignalGuard — Data Documentation

All files in this directory are sourced from public government APIs and databases. No data has been simulated or fabricated.

---

## export/ — Analysis Files

### `signalguard_priority_scores_miami_dade.csv` ← MAIN OUTPUT
**1,572 rows × 21 columns** — One row per FDOT traffic signal in Miami-Dade County with all raw inputs, normalized scores, final priority score, tier assignment, and source attribution.

| Column | Type | Description |
|--------|------|-------------|
| `signal_id` | int | FDOT signal identifier |
| `lat` | float | Signal latitude (WGS84) |
| `lon` | float | Signal longitude (WGS84) |
| `county` | str | County name — Miami-Dade |
| `aadt` | int | Annual Average Daily Traffic (vehicles/day) from nearest FDOT road segment |
| `closest_storm_km` | float | Haversine distance to nearest HURDAT2 track point (km) |
| `storm_count_250km` | int | Number of HURDAT2 storms with any track point within 250 km (context only — not used in scoring) |
| `aadt_score` | float | Min-max normalized AADT ∈ [0, 1] |
| `raw_exposure` | float | 1 / closest_storm_km (pre-normalization exposure value) |
| `hurr_score` | float | Min-max normalized storm exposure ∈ [0, 1] |
| `priority_score` | float | 0.5 × aadt_score + 0.5 × hurr_score ∈ [0, 1] |
| `tier` | str | HIGH / MODERATE / LOW (percentile-based) |
| `noaa_storm_events` | int | County-level NOAA storm events count (Miami-Dade, 2014–2022) — same for all signals |
| `fema_declarations` | int | County-level FEMA disaster declarations (Miami-Dade, 2016–2022) — same for all signals |
| `source_signals` | str | FDOT ArcGIS REST API — Traffic Signal Locations TDA |
| `source_aadt` | str | FDOT ArcGIS REST API — Annual Average Daily Traffic TDA (2025) |
| `source_hurricane` | str | NOAA HURDAT2 Atlantic Hurricane Database (1851–2023) |
| `source_storm_events` | str | NOAA Storm Events Database (CZ_FIPS=86, 2014–2022) |
| `source_fema` | str | FEMA OpenFEMA Disaster Declarations API (2016–2022) |
| `query_date` | str | Date data was queried — April 2026 |

---

### `fdot_traffic_signals_miami_dade.csv`
**1,572 rows** — Raw FDOT signal inventory for Miami-Dade County as queried from the FDOT ArcGIS REST API in April 2026.

- **Source**: FDOT ArcGIS REST API — Traffic Signal Locations TDA
- **API endpoint**: `https://services1.arcgis.com/O1JpcwDW8sjYuddV/arcgis/rest/services/Traffic_Signal_Locations_TDA/FeatureServer/0/query`
- **Year note**: No year field exists in the source. Represents current FDOT inventory as of query date (April 2026).

---

### `fdot_aadt_miami_dade.csv`
**1,848 rows** — Raw FDOT Annual Average Daily Traffic road segments for Miami-Dade County.

- **Source**: FDOT ArcGIS REST API — Annual Average Daily Traffic TDA
- **API endpoint**: `https://services1.arcgis.com/O1JpcwDW8sjYuddV/arcgis/rest/services/Annual_Average_Daily_Traffic_TDA/FeatureServer/0/query`
- **Year**: 2025 AADT data

---

### `hurdat2_hurricane_tracks.csv`
**54,749 rows** — NOAA HURDAT2 Atlantic hurricane track points from 1,973 named storms.

- **Source**: NOAA National Hurricane Center — HURDAT2 Atlantic Hurricane Database
- **Download**: `https://www.nhc.noaa.gov/data/hurdat/hurdat2-1851-2023-051124.txt`
- **Period**: 1851–2023
- **Key columns**: storm_id, storm_name, datetime, lat, lon, max_wind_kt, min_pressure_mb, storm_type
- **Note**: This is the full Atlantic basin dataset — not filtered to Miami-Dade. Haversine distances were computed from all track points to each signal.

---

### `noaa_storm_events_miami_dade.csv`
**87 rows** — NOAA Storm Events filtered to Miami-Dade County (CZ_FIPS=86).

- **Source**: NOAA Storm Events Database
- **Portal**: `https://www.ncdc.noaa.gov/stormevents/` → State: FL, County: MIAMI-DADE
- **Bulk CSVs**: `https://www.ncei.noaa.gov/pub/data/swdi/stormevents/csvfiles/`
- **Period**: 2014–2022
- **Use in project**: County-level context only — not used in signal scoring formula. All 1,572 signals share the same county-level count (87).

---

### `fema_disaster_declarations_miami_dade.csv`
**10 rows** — FEMA disaster declarations for Miami-Dade County.

- **Source**: FEMA OpenFEMA Disaster Declarations Summaries API (v2)
- **API endpoint**: `https://www.fema.gov/api/open/v2/disasterDeclarationsSummaries?state=FL&designatedArea=MIAMI-DADE%20COUNTY`
- **Dataset catalog**: `https://www.fema.gov/about/openfema/data-sets`
- **Period**: 2016–2022
- **Use in project**: County-level context only — not used in signal scoring formula.

---

---

## Normalization Reference

| Component | Raw min | Raw max | Denominator used |
|-----------|---------|---------|-----------------|
| AADT (veh/day) | 1,500 | 271,000 | 269,500 |
| closest_storm_km | 0.15 km | 12.24 km | — |
| raw_exposure (1/km) | 0.0817 | 6.667 | 6.585 |

These values are computed from the 1,572 Miami-Dade signals only. Applying this framework to another county would require recomputing the normalization ranges from that county's signal set.
