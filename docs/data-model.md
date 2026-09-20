# Calmer Commutes Data Model

```
Version: 0.1
Last updated: 2026-09-16
Purpose: Define tables and fields for the project.
```

## `path`

- Traversable path between intersection nodes. Could be a road segment, path, etc. One row of data represents a homogeneous traversable segment.
- PK: path_id
- Links to [1, 2] intersections; potentially more, need to explore
- Fields: width, length, paved, lanes, bike lane, one-way, traffic type (car, ped, mixed), traffic level, speed limit, zoning
- Changes in these values should result in a transition point and new path.
- Consider splitting time-dependent variables into separate tables.

## `intersection`

- Named intersections, or meaningful transition between paths. One row represents a physical crossing or transition.
- Each intersection must be uniquely geolocated.
- PK: intersection_id
- Links to [1, ...] paths (one to many)
- Fields: lat, lon, type, signaling, # connected paths (aggregated from join), wait time, traffic type, traffic level
- Consider splitting time-dependent variables into separate tables.

## `bike_facility`

- Bike infrastructure such as dedicated paths or lanes.
- PK: facility_id
- Links to path or intersection
- Fields: lat, lon, path_id, width, type, safety

## `bike_poi`
- Bike points of interest such as stations or racks (static)
- Fields: lat, lon, type, parking spots, cleanliness

## `crash_event`

- Crashes tied to specific paths or intersections. One row represents a single incident.
- PK: crash_id
- Links to path, intersection (or both)
- Fields: lat, lon, date, time, path_id, intersection_id, vehicle types, # involved, severity, ambulance, medical outcomes

## `path_score`

- Weighted path quality score, plus components. One row represents a score at a given time.
- Consider time-varying scores and a way to designate history vs. current data.
- PK: score_id
- Links to path
- Fields: path_id, time_id, weighted_score, traffic_score, speed_score, intersection_score, scenery_score, current_ind, date_added