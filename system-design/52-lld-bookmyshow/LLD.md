# LLD: BookMyShow & Concurrency Handling

## 1. Requirements

- **Theatres, screens, shows** (movie + time slot); **seats** per show (numbered, can be locked/available/booked).
- **User:** Browse shows → select seats → **lock** seats (temporary, e.g. 5 min) → pay → **confirm** booking; or release lock on timeout/cancel.
- **Concurrency:** Two users must not book the same seat; **lock** before payment and **confirm** after payment; release lock on timeout or failure.
- **Optional:** Multiple screens, seat categories (gold/silver), waitlist.

---

## 2. Core Classes

```text
Theatre, Screen, Show – showId, movieId, screenId, startTime
Seat – seatId, row, number, category; status (AVAILABLE/LOCKED/BOOKED)
ShowSeat – showId, seatId, status, lockedAt?, lockedByUserId?
BookingService – getAvailableSeats(showId); lockSeats(showId, seatIds, userId) → lockId | failure; confirmBooking(lockId, paymentId) → booking; releaseLock(lockId)
Lock – lockId, showId, seatIds[], userId, expiresAt
Booking – bookingId, userId, showId, seatIds[], status (CONFIRMED/CANCELLED)
```

---

## 3. Concurrency: Lock Then Confirm

1. **Lock:** User selects seats; BookingService.lockSeats(showId, seatIds, userId). For each seat: **conditional update** – UPDATE show_seat SET status=LOCKED, locked_at=now(), locked_by=userId WHERE show_id=? AND seat_id IN (?) AND status=AVAILABLE. If affected rows < len(seatIds), some seat already taken → return failure. Else create Lock(lockId, ...); return lockId; set TTL (e.g. 5 min) to auto-release.
2. **Confirm:** User pays; BookingService.confirmBooking(lockId, paymentId). Validate lock exists and not expired; UPDATE show_seat SET status=BOOKED WHERE ... AND locked_by=userId (and seat in lock); INSERT booking; DELETE lock; return booking.
3. **Release:** On timeout (cron or TTL callback): set seats back to AVAILABLE; delete lock. On user cancel before pay: same release.

---

## 4. Avoiding Double Booking (DB)

- **Optimistic:** Read seats; check all AVAILABLE; update with WHERE status=AVAILABLE; if row count mismatch, retry or fail.
- **Pessimistic:** SELECT FOR UPDATE on selected rows (or on show row) so other transactions block; then update and commit.
- **Unique constraint:** (show_id, seat_id) unique; only one row per seat per show; status transition AVAILABLE→LOCKED→BOOKED; one update per seat so first writer wins.

---

## 5. Key Methods

```text
BookingService.getAvailableSeats(showId) → List<Seat>
BookingService.lockSeats(showId, seatIds, userId) → lockId | null
BookingService.confirmBooking(lockId, paymentId) → Booking
BookingService.releaseLock(lockId)
LockManager.releaseExpiredLocks()   // cron every 1 min
```

---

## 6. Design Patterns

- **State:** Seat status (Available, Locked, Booked); transition rules.
- **Strategy:** Optional pricing by seat category and show time.
- **Facade:** BookingService as facade over SeatRepository, LockRepository, PaymentService.

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

