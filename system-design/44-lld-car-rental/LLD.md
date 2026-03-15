# LLD: Car Rental System

## 1. Requirements

- **Vehicles** (id, type, model, dailyRate); **availability** by date (not booked).
- **Booking:** User selects vehicle and date range; **reserve** (lock); **confirm** (pay); **pickup** and **return** (record actual dates); **billing** (days × rate + optional extras).
- **States:** Vehicle (AVAILABLE, RESERVED, RENTED, MAINTENANCE); Booking (PENDING, CONFIRMED, PICKED_UP, RETURNED, CANCELLED).
- **Optional:** Damage report, late return fee, different locations for pickup/return.

---

## 2. Core Classes

```text
Vehicle – id, type, model, dailyRate; status (AVAILABLE/RESERVED/RENTED/MAINTENANCE)
Booking – id, vehicleId, userId, startDate, endDate, status; pickupAt?, returnAt?
InventoryService – isAvailable(vehicleId, start, end); reserve(vehicleId, start, end) → bookingId; release(bookingId)
BillingService – calculate(booking) → amount (days × rate + extras); applyLateFee(actualReturn - endDate)
```

---

## 3. Flow

1. **Search:** getAvailableVehicles(start, end) → vehicles where no booking overlaps [start, end].
2. **Reserve:** Create Booking(PENDING); mark vehicle RESERVED for [start, end]; TTL (e.g. 30 min) to pay.
3. **Confirm:** On payment; Booking → CONFIRMED.
4. **Pickup:** Booking → PICKED_UP; vehicle → RENTED; record pickup time.
5. **Return:** Booking → RETURNED; vehicle → AVAILABLE; record return time; bill = (actual days) × rate + late fee if any.
6. **Cancel:** Release vehicle; Booking → CANCELLED; refund per policy.

---

## 4. Design Patterns

- **State:** VehicleStatus, BookingStatus; transitions on reserve, confirm, pickup, return.
- **Factory:** VehicleFactory by type (Economy, SUV, etc.) for pricing/creation.
- **Strategy:** PricingStrategy (weekday vs weekend, long-term discount).

---

## 5. Key Methods

```text
InventoryService.getAvailableVehicles(start, end)
BookingService.reserve(vehicleId, userId, start, end) → bookingId
BookingService.confirm(bookingId, paymentId)
BookingService.pickup(bookingId), return(bookingId)
BillingService.calculate(bookingId) → amount
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

