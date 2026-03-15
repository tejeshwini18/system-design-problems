# LLD: Restaurant Management System

## 1. Requirements

- **Restaurant:** tables (capacity, status: AVAILABLE/OCCUPIED/RESERVED); **menu** (items, price, category).
- **Order:** tableId, list of (menuItemId, qty); status (CREATED/PREPARING/SERVED/PAID); **bill** (subtotal, tax, tip).
- **Workflow:** Waiter creates order → kitchen receives → prepared → served → customer pays → table freed.
- **Reservation:** Customer reserves table for date/time slot; table marked reserved; no-show handling optional.
- **Optional:** Split bill; discounts; kitchen display; multiple branches.

---

## 2. Core Classes

```text
Table – tableId, capacity, status (AVAILABLE/OCCUPIED/RESERVED)
MenuItem – itemId, name, price, category
Order – orderId, tableId, items: List<OrderItem>, status, createdAt
OrderItem – menuItemId, quantity, priceAtOrder
Reservation – tableId, customerId, slotStart, slotEnd, status
RestaurantService – createOrder(tableId, items); updateOrderStatus(orderId, status); getBill(orderId); payBill(orderId, amount); reserveTable(tableId, slot); freeTable(tableId)
```

---

## 3. Order Flow

1. createOrder(tableId, items): Validate table OCCUPIED or mark OCCUPIED; create Order with status CREATED; return orderId.
2. Kitchen: updateOrderStatus(orderId, PREPARING) then SERVED.
3. getBill(orderId): Sum (qty * price) per item; add tax; return total.
4. payBill(orderId, amount): Update order status PAID; mark table AVAILABLE (or trigger freeTable).
5. freeTable(tableId): Set table status AVAILABLE.

---

## 4. Reservation

- reserveTable(tableId, slotStart, slotEnd): Check table not already reserved for overlapping slot; create Reservation; set table RESERVED for that slot (or keep AVAILABLE and check on arrival). On arrival: mark table OCCUPIED; on no-show (after slot end): release.

---

## 5. Key Methods

```text
RestaurantService.createOrder(tableId, items) → orderId
RestaurantService.getBill(orderId) → Bill
RestaurantService.payBill(orderId, amount)
RestaurantService.reserveTable(tableId, customerId, slot)
Table.getStatus(), setStatus()
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

