# LLD: Ticketmaster

## APIs

```http
GET    /v1/events/:id/seats   Available seats
POST   /v1/events/:id/reserve   Body: { seat_ids[] }   → hold_id, expires_at
POST   /v1/holds/:id/confirm   Body: { payment_id }   → ticket_ids
POST   /v1/holds/:id/release
GET    /v1/tickets/:id
```

## Key Classes

```text
SeatService – getAvailableSeats(eventId); lockSeats(eventId, seatIds, userId) → holdId
Hold – holdId, eventId, seatIds[], userId, expiresAt
TicketService – confirmHold(holdId, paymentId) → tickets; releaseHold(holdId)
```

## Reserve (Steps)

1. BEGIN; SELECT * FROM seats WHERE event_id=? AND seat_id IN (?) AND status='AVAILABLE' FOR UPDATE; if count < len(seatIds), ROLLBACK return error.
2. UPDATE seats SET status='LOCKED', locked_by=?, expires_at=? WHERE ...; INSERT holds; COMMIT; return holdId.
3. Cron: UPDATE seats SET status='AVAILABLE' WHERE status='LOCKED' AND expires_at < NOW().
