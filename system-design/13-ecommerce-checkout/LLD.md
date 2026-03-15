# Low-Level Design: E-commerce Checkout System

## 1. APIs

```http
GET    /v1/cart                         Get current cart
PUT    /v1/cart/items                   Add/update item (sku, quantity)
DELETE /v1/cart/items/:sku              Remove item
POST   /v1/cart/merge                   Merge guest cart (guest_cart_id) after login

GET    /v1/checkout/summary              Get checkout summary (cart + address + shipping + coupons → totals)
POST   /v1/checkout/summary              Set address, shipping method, coupon; return updated summary

POST   /v1/orders
  Headers: Idempotency-Key: <key>
  Body: { "cart_id": "...", "address_id": "...", "payment_method_id": "...", "coupon_code": "..." }
  Response: { "order_id": "...", "status": "placed" | "paid", "total", ... }

GET    /v1/orders/:id                   Get order details
POST   /v1/orders/:id/cancel           Cancel order (if not shipped)
GET    /v1/orders                      List my orders (paginated)
```

---

## 2. Database Schemas

```sql
carts (cart_id PK, user_id FK NULL, updated_at);
cart_items (cart_id FK, sku FK, quantity INT, PRIMARY KEY (cart_id, sku));

orders (order_id PK, user_id FK, status ENUM(placed, paid, shipped, delivered, cancelled, payment_failed),
  idempotency_key UNIQUE, total_cents BIGINT, currency, address_snapshot JSON, created_at);
order_items (order_id FK, sku, quantity INT, unit_price_cents BIGINT, PRIMARY KEY (order_id, sku));

inventory (sku PK, available_quantity INT, updated_at);
reservations (reservation_id PK, order_id FK, sku, quantity INT, expires_at, created_at);
CREATE INDEX idx_reservations_expires ON reservations(expires_at);

idempotency (idempotency_key PK, order_id FK, response_body JSON, created_at);
```

---

## 3. Key Classes / Modules

```text
CartService
  - getCart(userIdOrGuestId) → cart with items
  - addItem(cartId, sku, quantity)
  - removeItem(cartId, sku)
  - mergeCart(guestCartId, userId)

CheckoutService
  - getSummary(cartId, addressId?, shippingId?, couponCode?) → summary (items, subtotal, discount, tax, shipping, total)
  - validateCart(cartId) → valid | errors[]

OrderService
  - placeOrder(userId, idempotencyKey, cartId, addressId, paymentMethodId, couponCode?) → order
  - getOrder(orderId, userId) → order
  - cancelOrder(orderId, userId)

InventoryService
  - reserve(sku, quantity, orderId, ttlMinutes) → reservationId | throw OutOfStock
  - release(reservationId) or releaseByOrderId(orderId)
  - checkAvailable(sku, quantity) → boolean

PricingService
  - getTotals(cartItems, addressId, shippingId, couponCode?) → { subtotal, discount, tax, shipping, total }
```

---

## 4. Place Order (Steps)

1. Get idempotency key from header; lookup in idempotency table. If found, return stored response (same order_id and status).
2. Load cart; validate (all items exist, in stock at check time). Get checkout summary (pricing).
3. Begin transaction (or saga):
   a. For each cart item: InventoryService.reserve(sku, qty, orderId, 15). If any fails, rollback previous reserves, return 409.
   b. Insert order (status=placed), order_items; commit.
4. Call PaymentService.capture(paymentMethodId, total, idempotencyKey). On success: update order status=paid; store idempotency → order_id + response. On failure: release all reservations for this order; update order status=payment_failed; store idempotency → error response; return 402.
5. Send confirmation event (email/notification). Return 200 with order.

---

## 5. Inventory Reserve (Atomic)

```sql
-- Reserve: create reservation and decrement available
INSERT INTO reservations (reservation_id, order_id, sku, quantity, expires_at)
VALUES (?, ?, ?, ?, NOW() + INTERVAL 15 MINUTE);
UPDATE inventory SET available_quantity = available_quantity - ? WHERE sku = ? AND available_quantity >= ?;
-- If update affected 0 rows, rollback insert, return OutOfStock
```

Alternatively: single UPDATE with subquery to avoid race (reserve in one statement).

---

## 6. Reservation Release (Background Job)

- Every 1 min: SELECT * FROM reservations WHERE expires_at < NOW() AND order_id IN (SELECT order_id FROM orders WHERE status = 'placed' AND created_at < NOW() - 15 min).
- For each: increment inventory; delete reservation; optionally update order to expired.

---

## 7. Error Handling

- Cart empty: 400.
- Item out of stock: 409 with list of unavailable SKUs.
- Invalid coupon: 400; exclude from summary.
- Payment declined: 402; order in payment_failed; user can retry with different method (new request with new idempotency key).
- Duplicate idempotency key: return same response as first request (200 with order or 402 with same error).
