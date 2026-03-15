# LLD: Hotel Booking

## APIs

```http
GET    /v1/hotels/search?city=...&check_in=...&check_out=...&guests=...
POST   /v1/bookings   Body: { hotel_id, room_type_id, check_in, check_out, guest_id }   → booking_id (pending)
POST   /v1/bookings/:id/confirm   Body: { payment_id }
POST   /v1/bookings/:id/cancel
```

## Key Classes

```text
InventoryService – getAvailable(hotelId, roomTypeId, checkIn, checkOut) → count, price
BookingService – reserve(...) → bookingId; confirm(bookingId, paymentId); cancel(bookingId)
InventoryService – reserve(roomTypeId, checkIn, checkOut, count); release(bookingId)
```

## Availability Query

For each date in [checkIn, checkOut): available = total - booked. Min(available) over dates >= roomsNeeded; price = sum(rate[date]).

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

