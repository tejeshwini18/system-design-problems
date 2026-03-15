# LLD: Food Delivery

## APIs

```http
GET    /v1/restaurants?lat=...&lon=...&cuisine=...
GET    /v1/restaurants/:id/menu
POST   /v1/orders   Body: { restaurant_id, items[], address, payment_method }   → order_id
GET    /v1/orders/:id   Status, ETA, tracking
POST   /v1/orders/:id/cancel
POST   /v1/restaurant/orders/:id/accept   (restaurant)
POST   /v1/restaurant/orders/:id/ready
POST   /v1/delivery/orders/:id/accept   (partner)
POST   /v1/delivery/orders/:id/delivered
```

## Order State Machine

PENDING → ACCEPTED (restaurant) / REJECTED / CANCELLED  
ACCEPTED → PREPARING → READY  
READY → ASSIGNED (partner) → OUT_FOR_DELIVERY → DELIVERED  
Cancel allowed in PENDING, ACCEPTED (with policy).

## Key Classes

```text
OrderService – create(cart, address, payment) → orderId; getStatus(orderId); cancel(orderId)
RestaurantOrderService – accept(orderId); setReady(orderId)
DispatchService – assignPartner(orderId) → partnerId; partnerAccept(orderId); markDelivered(orderId)
TrackingService – updateLocation(orderId, lat, lon); getETA(orderId)
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

