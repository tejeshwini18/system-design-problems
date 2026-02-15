# LLD: Vending Machine

## 1. Requirements

- **Machine** has **slots** (rows × columns); each slot holds one **item** (product + price) and **quantity**.
- **User:** Select slot (e.g. A1); insert coins/cash; machine dispenses item and change.
- **States:** Idle → ProductSelected → MoneyInserted → Dispense → Idle (or Refund on cancel).
- **Payment:** Accept coins (e.g. 1, 5, 10); or notes; total inserted ≥ price then dispense and return change.
- **Inventory:** Track quantity per slot; "Sold out" if quantity = 0.
- **Optional:** Admin to refill inventory and collect cash.

---

## 2. Core Classes

```text
Product – id, name, price
Slot – slotId (e.g. "A1"), product, quantity; decrementQuantity(), isAvailable()
VendingMachine – slots[][], currentState, selectedSlot, insertedAmount
VendingMachineState (State pattern) – IdleState, ProductSelectedState, MoneyInsertedState, DispenseState
CoinInventory – map<denomination, count>; add(denomination), getChange(amount) → list of coins, canDispenseChange(amount)
```

---

## 3. State Flow

1. **Idle:** User presses slot (e.g. A1). Validate slot exists and quantity > 0; set selectedSlot; transition to **ProductSelected**; display price.
2. **ProductSelected:** User inserts coin; insertedAmount += value; if insertedAmount >= price, transition to **MoneyInserted** (or stay and show "Add more"). User can cancel → refund insertedAmount → Idle.
3. **MoneyInserted:** Machine dispenses product (selectedSlot.decrementQuantity()); calculate change = insertedAmount - price; dispense change from CoinInventory; transition to **Dispense** then **Idle**. If change cannot be given, refund and show message → Idle.
4. **Dispense:** Release item to tray; release change; clear selectedSlot and insertedAmount; → Idle.

---

## 4. Change Calculation

- **Greedy:** For change amount, use largest denomination first (e.g. 10, 5, 1); subtract from CoinInventory. If not possible (e.g. no 1s), return "Exact change only" and refund.
- **CoinInventory:** Maintain count per denomination; getChange(amount) returns list of coins and deducts; if cannot make change, return null and don’t deduct.

---

## 5. Design Patterns

- **State:** VendingMachineState and subclasses; each state handles: selectProduct, insertCoin, dispense, cancel.
- **Strategy:** Optional different payment (card vs cash) via PaymentStrategy.
- **Factory:** ProductFactory for creating product types.

---

## 6. Key Methods

```text
VendingMachine.selectSlot(slotId)
VendingMachine.insertCoin(denomination)
VendingMachine.cancel()   // refund
VendingMachine.dispense()   // internal after payment valid
Slot.decrementQuantity(), getQuantity(), getProduct()
CoinInventory.getChange(amount) → List<Coin> | null
```
