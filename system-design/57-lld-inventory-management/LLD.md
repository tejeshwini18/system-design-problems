# LLD: Inventory Management System

## 1. Requirements

- **Products:** productId, name, SKU; **stock** (quantity); **warehouse** or location optional.
- **Operations:** **Add stock** (purchase, restock), **Reserve** (for order), **Deduct** (on shipment/cancel), **Release** (cancel reservation).
- **Concurrency:** Oversell prevention; reserve before payment; deduct on confirm; release on cancel or timeout.
- **Optional:** Multiple warehouses; reorder threshold; audit log; batch/serial numbers.

---

## 2. Core Classes

```text
Product – productId, name, sku
Inventory – productId, warehouseId?, quantityAvailable, quantityReserved
  available = quantityAvailable - quantityReserved
Reservation – reservationId, productId, quantity, orderId?, expiresAt
InventoryService – addStock(productId, qty); reserve(productId, orderId, qty) → reservationId; deduct(productId, orderId, qty); release(reservationId) or releaseByOrderId(orderId)
```

---

## 3. Reserve / Deduct / Release

- **Reserve:** UPDATE inventory SET quantity_reserved = quantity_reserved + ? WHERE product_id = ? AND (quantity_available - quantity_reserved) >= ?; if affected = 0, fail (insufficient). INSERT reservation; return reservationId. **TTL:** Cron to release reservations where expires_at < now() and decrement quantity_reserved.
- **Deduct (confirm order):** Find reservation by orderId; UPDATE inventory SET quantity_available = quantity_available - ?, quantity_reserved = quantity_reserved - ? WHERE product_id = ?; DELETE reservation.
- **Release (cancel):** Same as deduct but only quantity_reserved -= ? and quantity_available unchanged; DELETE reservation.

---

## 4. Key Methods

```text
InventoryService.addStock(productId, warehouseId?, qty)
InventoryService.reserve(productId, orderId, qty) → reservationId
InventoryService.confirmAndDeduct(orderId)   // on payment success
InventoryService.releaseReservation(reservationId)
InventoryService.getAvailableQuantity(productId) → int
```

---

## 5. Design Patterns

- **State:** Reservation (Active, Expired, Confirmed, Released). **Strategy:** Allocation strategy (FIFO, by warehouse).
