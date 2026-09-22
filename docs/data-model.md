# Calmer Commutes Data Model

```
Version: 0.1
Last updated: 2026-09-20
Purpose: Define tables and fields for the project.
```

## `path`

- Traversable path between intersection nodes. Could be a road segment, path, etc. One row of data represents a homogeneous traversable segment.
- Source: OpenStreetMap, Open Data Naperville
- PK: path_id
- Links to [1, 2] intersections; potentially more, need to explore
- Changes in field values should result in a transition point and new path.
- Consider splitting time-dependent variables into separate tables.

| Name | Source | Updates | DQM |
|---|---|---|---|
| width | OSM | static |  |
| length | OSM | static |  |
| paved | OSM | rare |  |
| lanes | OSM | occasional | construction impact |
| bike lane | OSM | rare | source updates from? |
| one-way | OSM | static | potential construction impact |
| traffic type | OSM | occasional | car, ped, mixed, missing |
| traffic level | ODN? | frequent | time-of-day changes |
| speed limit | OSM | occasional | time-of-day changes |
| zoning | ODN? | static |  |

## `intersection`

- Named intersections, or meaningful transition between paths. One row represents a physical crossing or transition.
- Source: OpenStreetMap, Open Data Naperville
- Each intersection must be uniquely geolocated.
- PK: intersection_id
- Links to [1, ...] paths (one to many)
- Consider splitting time-dependent variables into separate tables.

| Name | Source | Updates | DQM |
|---|---|---|---|
| lat | OSM | static | verify geocoding |
| lon | OSM | static | verify geocoding |
| type | OSM | rare | potential construction impact or closures |
| signaling | OSM | rare |  |
| # connected paths | `path` | rare | recalculate from link table |
| wait time | OSM? | frequent | time-of-day dependent |
| traffic type | OSM or ODN | occasional | potential construction impact, closures, time of day |
| traffic level | OSM? | frequent | time-of-day dependent |

## `bike_facility`

- Bike infrastructure such as dedicated paths or lanes.
- Source: OpenStreetMap?, search for bike dataset
- PK: facility_id
- Links to path or intersection

| Name | Source | Updates | DQM |
|---|---|---|---|
| lat | OSM | static | verify geocoding |
| lon | OSM | static | verify geocoding |
| path_id | derived from `path` | static |  |
| width | OSM | rare |  |
| type | OSM? | rare |  |
| safety | derived from `crash_event` | occasional | how to handle missing values? |

## `bike_poi`

- Bike points of interest such as stations or racks (static)
- Source: OpenStreetMap?, search for bike dataset

| Name | Source | Updates | DQM |
|---|---|---|---|
| lat | OSM | static | verify geocoding |
| lon | OSM | static | verify geocoding |
| type | OSM? | rare |  |
| parking spots | bike data | rare |  |
| cleanliness | bike data | occasional |  |

## `crash_event`

- Crashes tied to specific paths or intersections. One row represents a single incident.
- Source: Open Data Naperville, check IDOT crash data
- PK: crash_id
- Links to path, intersection (or both)
- Rare updates below mean check for report updates X months after incident?

| Name | Source | Updates | DQM |
|---|---|---|---|
| lat | IDOT | static | verify geocoding |
| lon | IDOT | static | verify geocoding |
| date | IDOT | static |  |
| time | IDOT | static |  |
| path_id | derived from `path` | static |  |
| intersection_id | derived from `intersection` | static |  |
| vehicle types | IDOT | rare | recheck schedule? |
| # involved | IDOT | rare | recheck schedule? |
| severity | IDOT | rare | recheck schedule? |
| ambulance | IDOT | rare | recheck schedule? |
| medical outcomes | IDOT? | rare | recheck schedule? |

## `path_score`

- Weighted path quality score, plus components. One row represents a score at a given time.
- Source: derived from other datasets
- Consider time-varying scores and a way to designate history vs. current data.
- PK: score_id
- Links to path

| Name | Source | Updates | DQM |
|---|---|---|---|
 path_id | derived from `path` | static |  |
| time_id | secondary key | static | use for time of day; potentially historical scoring? |
| weighted_score | derived from other scores | frequently | potentially refresh upon path call |
| traffic_score | derived from `path` or `intersection` | frequent | potentially time dependent |
| speed_score | derived from `path` | occasional | potentially time dependent |
| intersection_score | derived from `intersection` |  | potentially time dependent |
| scenery_score | derived from `path` | rare |  |
| current_ind | computed on commit | occasional | need a lifecycle plan |
| date_added | computed on commit | static | potential historic score tracker |