# LLD: Pizza Billing System

## 1. Requirements

- **Pizza:** Base (size: S/M/L) with base price; **toppings** (extra cost each); total = base + sum(toppings).
- **Order:** One or more pizzas; optional discount (coupon); tax; total.
- **Optional:** Different crust types; combo deals; delivery fee.

---

## 2. Core Classes (Decorator for Toppings)

```text
Pizza (abstract or interface) – getDescription(), getCost()
  MargheritaPizza(size) – base cost by size
  ToppingDecorator extends Pizza – wrapped: Pizza; getCost() = wrapped.getCost() + toppingPrice; getDescription() = wrapped.getDescription() + ", ToppingName"

Concrete toppings: CheeseDecorator(pizza), OliveDecorator(pizza), etc.

Order – items: List<Pizza>; addPizza(pizza); getSubtotal(); applyCoupon(code); getTax(); getTotal()
PricingStrategy – getBasePrice(size); getToppingPrice(toppingName)
```

---

## 3. Builder Alternative

```text
Pizza.Builder()
  .size(LARGE)
  .addTopping("cheese")
  .addTopping("olive")
  .build() → Pizza
```
- Builder sets base + list of toppings; build() creates one object (e.g. BasePizza + toppings list); cost = basePrice(size) + sum(toppingPrices).

---

## 4. Cost Calculation

- **Decorator:** pizza.getCost() recursively: base.getCost() + topping.getCost().
- **Or:** Pizza has size and List<String> toppings; cost = getBasePrice(size) + sum(getToppingPrice(t)) for t in toppings.
- **Order:** subtotal = sum(pizza.getCost()); discount = coupon.apply(subtotal); tax = (subtotal - discount) * taxRate; total = subtotal - discount + tax.

---

## 5. Design Patterns

- **Decorator:** Add toppings dynamically; each topping wraps a Pizza and adds cost and description.
- **Builder:** Fluent construction of Pizza with multiple toppings.
- **Strategy:** Optional PricingStrategy (weekday vs weekend prices).

---

## 6. Key Methods

```text
Pizza.getCost(), getDescription()
Order.addPizza(pizza), getSubtotal(), applyCoupon(code), getTotal()
```
