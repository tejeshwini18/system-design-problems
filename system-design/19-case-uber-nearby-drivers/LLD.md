# Low-Level Design: Uber Nearby Drivers at 1M RPS

## 1. APIs

```http
POST   /v1/drivers/:id/location      Driver: update location (lat, lng, status)
  Body: { "lat": 37.77, "lng": -122.41, "status": "available" }

GET    /v1/nearby?lat=37.77&lng=-122.41&radius_km=5&limit=20
  Response: { "drivers": [ { "driver_id": "...", "lat": 37.77, "lng": -122.42, "distance_km": 0.5 } ] }
```

---

## 2. Redis Key Design (GEOSPATIAL)

```text
Key per region:  drivers:geo:{region_id}   e.g. drivers:geo:sf
Type: Redis GEOSPATIAL (sorted set under the hood)
  GEOADD drivers:geo:sf -122.41 37.77 driver_123
  GEORADIUS drivers:geo:sf -122.41 37.77 5 km WITHDIST COUNT 50 ASC

Available-only key (set):  drivers:available:{region_id}  → set of driver_ids
  SADD drivers:available:sf driver_123
  SREM drivers:available:sf driver_123   // when on trip

Query flow: GEORADIUS → list of (driver_id, dist); SINTER with drivers:available:sf (in app: filter by membership); return.
```

---

## 3. Region / Partition Resolution

```text
region_id = getRegionId(lat, lng)   // e.g. geohash prefix, or city polygon lookup
  - Precomputed grid: (lat,lng) → region_id
  - Or: call a lightweight geo service / lookup table
```

- Location update: driver’s region from last known or from (lat, lng); if region changed, remove from old region key and add to new (GEOADD to new; ZREM or separate “members” set for old).
- Query: region_id from (lat, lng); query that region’s key only.

---

## 4. Key Classes / Modules

```text
LocationIngestion
  - updateLocation(driverId, lat, lng, status)
  - resolveRegion(lat, lng) → regionId
  - geoAdd(regionId, driverId, lng, lat)
  - setAvailable(regionId, driverId, available: boolean)

GeoService
  - getNearby(lat, lng, radiusKm, limit) → drivers[]
  - getRegionId(lat, lng) → regionId
  - geoRadius(regionId, lng, lat, radiusKm, limit) → [(driverId, distance)]
  - filterAvailable(regionId, driverIds) → driverIds
```

---

## 5. Grid-Based Alternative (Pseudocode)

```text
cell_size = 0.01   // ~1 km
cell_id(lat, lng) = (floor(lat / cell_size), floor(lng / cell_size))

// Driver update
old_cell = driver_cell[driver_id]
new_cell = cell_id(lat, lng)
if old_cell != new_cell:
  cells[old_cell].remove(driver_id)
  cells[new_cell].add(driver_id)
  driver_cell[driver_id] = new_cell
driver_location[driver_id] = (lat, lng)
driver_available[driver_id] = (status == "available")

// Query
center_cell = cell_id(lat, lng)
radius_cells = cells_overlapping_circle(lat, lng, radius_km)
candidates = []
for cell in radius_cells:
  for d in cells[cell]:
    if driver_available[d]:
      dist = haversine(lat, lng, driver_location[d])
      if dist <= radius_km: candidates.append((d, dist))
candidates.sort(by=dist)
return candidates[:limit]
```

---

## 6. Handling Region Change (Driver Moves City)

- On update: compute new region from (lat, lng). If different from current region stored for driver: remove driver from old region’s GEO set (maintain a mapping driver_id → current_region); GEOADD to new region; update driver_id → region_id map.
- Use a small hash key: driver:region:{driver_id} = region_id for fast lookup on update.

---

## 7. Error Handling and Limits

- **Invalid lat/lng:** 400.
- **Radius too large:** Cap (e.g. 50 km) to avoid scanning entire region.
- **Limit:** Cap (e.g. 100) to avoid large responses.
- **Redis down:** 503; circuit breaker; optional fallback to DB with bounding box query (slower).

---

## Interview-Readiness Enhancements

### API and consistency
- Mark idempotency requirements for mutation APIs.
- Specify pagination/cursor strategy for list endpoints.
- Clarify consistency guarantees per endpoint/workflow.

### Data model and concurrency
- Explicitly list partition key/index choices and why.
- State optimistic vs pessimistic locking policy and conflict handling.
- Define deduplication/idempotent-consumer strategy for async paths.

### Reliability and operations
- Add explicit failure scenarios with mitigations and degradation behavior.
- Add monitoring/alert thresholds for critical flows and queue lag.
- Document rollout and rollback steps for schema/API changes.

### Validation checklist
- Include unit + integration + load + failure-injection test cases for critical paths.

