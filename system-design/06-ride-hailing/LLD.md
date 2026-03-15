# Low-Level Design: Ride-Hailing System

## 1. APIs

### Rider
```http
POST   /v1/trips                    Create trip (pickup_lat, pickup_lng, drop_lat, drop_lng, type)
GET    /v1/trips/:id                Trip status, driver info, ETA
POST   /v1/trips/:id/cancel         Cancel trip
GET    /v1/trips/:id/eta            Get ETA to pickup / to drop
GET    /v1/trips/history            List past trips (paginated)
POST   /v1/trips/:id/rate           Rate driver (1-5, optional comment)
```

### Driver
```http
POST   /v1/driver/online            Go online (lat, lng)
POST   /v1/driver/offline           Go offline
PUT    /v1/driver/location          Update location (lat, lng) - or WebSocket
POST   /v1/trips/:id/accept         Accept trip
POST   /v1/trips/:id/decline        Decline trip
POST   /v1/trips/:id/start          Start trip (at pickup)
POST   /v1/trips/:id/complete       Complete trip (at drop)
GET    /v1/driver/trips             Current and recent trips
```

### WebSocket (driver location)
```text
WS /v1/driver/ws?token=<jwt>
Client → Server: { "type": "location", "lat": 12.34, "lng": 56.78 }
Server → Client: { "type": "trip_request", "trip": {...} }
```

---

## 2. Database Schemas

```sql
trips (
  trip_id PK, rider_id FK, driver_id FK NULL,
  pickup_lat, pickup_lng, drop_lat, drop_lng,
  status ENUM(searching, accepted, in_progress, completed, cancelled),
  fare_amount DECIMAL, fare_currency, surge_multiplier DECIMAL,
  created_at, accepted_at, started_at, completed_at, cancelled_at
);
CREATE INDEX idx_rider_status ON trips(rider_id, status, created_at DESC);
CREATE INDEX idx_driver_status ON trips(driver_id, status);

driver_locations (driver_id PK, lat, lng, updated_at)  -- or in Redis
```

---

## 3. Key Classes / Modules

```text
TripService
  - createTrip(riderId, pickup, drop, type) → tripId
  - getTrip(tripId) → trip
  - cancelTrip(tripId, userId)
  - acceptTrip(tripId, driverId)
  - startTrip(tripId)
  - completeTrip(tripId)

MatchingService
  - findNearbyDrivers(pickupLat, pickupLng, radiusKm, limit) → driverIds[]
  - dispatchTrip(tripId, driverIds[])  // push to drivers
  - onAccept(tripId, driverId)
  - onDecline(tripId, driverId)  // assign to next or re-search

LocationService
  - updateLocation(driverId, lat, lng)
  - getDriversInRadius(lat, lng, radiusKm) → driverIds[]

PricingService
  - calculateFare(tripId, pickup, drop, surgeMultiplier) → fare
  - getSurgeMultiplier(lat, lng) → multiplier
```

---

## 4. Trip State Machine

```text
searching → accepted (driver accepts)
         → cancelled (rider cancels or timeout)
accepted → in_progress (driver starts)
        → cancelled (rider/driver cancels)
in_progress → completed (driver completes)
completed → [terminal]
cancelled → [terminal]
```

---

## 5. Matching Algorithm (Steps)

1. Get pickup (lat, lng); compute geohash or grid cell.
2. Query LocationService.getDriversInRadius(pickup, radius=5km).
3. Filter: status=available, not already in trip.
4. Sort by distance (or rating); take top 5–10.
5. For each driver: send push "trip request" with trip details; start timeout (e.g. 15 s).
6. On first accept: assign driver to trip; notify others "request expired"; update trip state to accepted.
7. On all decline or timeout: expand radius or retry with next batch; after 3–5 attempts, mark "no drivers" and notify rider.

---

## 6. Idempotency and Concurrency

- Accept: use conditional update (trip.driver_id = NULL → set driver_id) so only one driver wins.
- Cancel: check state (only searching or accepted can be cancelled by rider).
- Location: last-write-wins; driver_id as key in Redis.

---

## 7. ETA Calculation

- To pickup: use distance(driver_location, pickup) / avg_speed (e.g. 25 km/h in city) or call mapping API (e.g. Google Maps).
- To drop: distance(pickup, drop) / avg_speed or mapping API. Cache or batch to reduce API cost.

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

