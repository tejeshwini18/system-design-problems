# LLD: Parking Lot System

## 1. Requirements

- **Parking lot** has multiple **floors**; each floor has **parking spots** (car, bike, truck).
- **Vehicle** types: Car, Bike, Truck (different spot sizes).
- **Park:** Assign first available spot of matching type; return ticket.
- **Unpark:** Given ticket, release spot; calculate fee (time-based: per hour or first N hours then per hour).
- **Optional:** Multiple pricing strategies (hourly, flat first 2h then hourly); display board (free spots per type).

---

## 2. Core Classes

```text
Vehicle (abstract) – numberPlate
  Car, Bike, Truck extend Vehicle

ParkingSpot (abstract) – id, floorId, type (CAR/BIKE/TRUCK), isAvailable
  CarSpot, BikeSpot, TruckSpot extend ParkingSpot

Floor – floorId, list of ParkingSpot; getAvailableSpot(vehicleType)

ParkingLot – singleton or single instance; list of Floor; park(vehicle), unpark(ticketId)

Ticket – ticketId, vehicle, spotId, floorId, startTime

PricingStrategy (interface) – calculate(entryTime, exitTime) → amount
  HourlyPricingStrategy, FlatThenHourlyPricingStrategy
```

---

## 3. Park Flow

1. Vehicle arrives; get vehicle type.
2. For each floor (or preferred order): spot = floor.getAvailableSpot(vehicleType). If spot != null, break.
3. If no spot, return "Parking full".
4. Mark spot occupied; create Ticket(ticketId, vehicle, spot, startTime); return ticket to user.
5. Store ticket in map (ticketId → Ticket) for unpark.

---

## 4. Unpark Flow

1. User gives ticketId; lookup Ticket.
2. exitTime = now; amount = pricingStrategy.calculate(entryTime, exitTime).
3. Mark spot available; remove ticket; return amount (and optionally receipt).

---

## 5. Display Board

- For each floor: count spots where type = X and isAvailable = true; show "Car: 5, Bike: 10, Truck: 2".
- Optional: observer pattern when spot freed/occupied to update board.

---

## 6. Design Patterns Used

- **Singleton:** ParkingLot (one lot).
- **Strategy:** PricingStrategy for different fee models.
- **Factory:** VehicleFactory.create(type), SpotFactory for spot types.
- **State:** Spot state (Available, Occupied) if modeled explicitly.

---

## 7. Key Methods

```text
ParkingLot.getInstance().park(vehicle) → Ticket | null
ParkingLot.getInstance().unpark(ticketId) → amount
Floor.getAvailableSpot(vehicleType) → ParkingSpot | null
PricingStrategy.calculate(entryTime, exitTime) → double
```

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

