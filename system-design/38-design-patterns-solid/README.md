# SOLID Principles & Design Patterns – Quick Reference

This folder covers **SOLID principles** and **GoF design patterns** with one example each, as commonly asked in LLD interviews.

---

## SOLID Principles

| Principle | Meaning | Example |
|-----------|--------|--------|
| **S** – Single Responsibility | One class, one reason to change | `OrderService` (create order) vs `OrderNotificationService` (send email) |
| **O** – Open/Closed | Open for extension, closed for modification | Add new payment type via `IPaymentStrategy` impl, no change to `CheckoutService` |
| **L** – Liskov Substitution | Subtypes must be substitutable for base type | `Square` extends `Rectangle` but don’t allow independent width/height if it breaks invariants |
| **I** – Interface Segregation | Many specific interfaces > one fat interface | `IFlyable`, `ISwimmable` instead of `IAnimal` with fly()+swim()+run() |
| **D** – Dependency Inversion | Depend on abstractions, not concretions | `OrderService` depends on `IInventoryService`, not `MySQLInventoryService` |

---

## Design Patterns (with typical LLD mapping)

| Pattern | Purpose | Example / Interview Topic |
|---------|---------|---------------------------|
| **Strategy** | Interchangeable algorithms | Payment: CardStrategy, UPIStrategy, WalletStrategy |
| **Observer** | One-to-many notify on state change | Notify-Me: User subscribes; on restock, notify all subscribers |
| **Decorator** | Add behavior without subclassing | Coffee: Espresso → MilkDecorator → WhipDecorator (cost, description) |
| **Factory** | Create object without exposing constructor | `VehicleFactory.create("car")` → Car instance |
| **Abstract Factory** | Families of related objects | GUIFactory → WinButton + WinTextBox or MacButton + MacTextBox |
| **Chain of Responsibility** | Pass request along chain of handlers | Validation: AuthHandler → RateLimitHandler → BusinessHandler |
| **Proxy** | Surrogate with same interface; control access | LazyLoadProxy, CacheProxy, AccessControlProxy |
| **Null Object** | Replace null with no-op object | `NullLogger` instead of `if (logger != null) logger.log()` |
| **State** | Behavior changes with internal state | Elevator: IdleState, MovingState, MaintenanceState |
| **Composite** | Tree of same interface (part-whole) | File System: File + Directory (contains File/Directory) |
| **Adapter** | Adapt incompatible interface | ThirdPartyPaymentAdapter wraps external API to our IPayment interface |
| **Singleton** | One instance globally | ConnectionPool, Logger, Config (thread-safe: double-check lock or enum) |
| **Builder** | Stepwise construction of complex object | `Pizza.Builder().size(L).addTopping("cheese").build()` |
| **Prototype** | Clone instead of new | Copy existing Document template; deep copy |
| **Bridge** | Decouple abstraction from implementation | Shape (abstract) + DrawingAPI (impl: Vector/Raster) |
| **Facade** | Simple interface to subsystem | BookingFacade: bookFlight + bookHotel + sendEmail in one call |
| **Flyweight** | Share state to save memory | TreeType (name, color) shared; Tree (x, y, type) per instance |
| **Command** | Encapsulate request as object | Undo/redo: Command with execute() + undo(); Invoker holds history |
| **Interpreter** | Interpret grammar (DSL) | Expression tree: Number, Add, Subtract; evaluate() |
| **Iterator** | Traverse collection without exposing internals | next(), hasNext(); for custom tree or composite |
| **Mediator** | Central object for peer communication | ChatRoom: users send to room; room broadcasts to others |
| **Memento** | Save/restore object state (undo) | Editor: save state to Memento; restore from Memento |
| **Template Method** | Skeleton in base class; steps overridden | DataParser: parse() calls read() → process() → write(); subclasses override steps |
| **Visitor** | Add ops to hierarchy without changing classes | AST: Node.accept(Visitor); Visitor.visit(Number), visit(Add) |

---

## File Index

- **[LLD: Notify-Me Button](39-lld-notify-me/LLD.md)** – Observer + subscription
- **[LLD: Pizza Billing](40-lld-pizza-billing/LLD.md)** – Decorator (toppings), Builder
- **[LLD: Parking Lot](41-lld-parking-lot/LLD.md)** – State, strategy (pricing)
- **[LLD: Snake & Ladder](42-lld-snake-ladder/LLD.md)** – Simple game state
- **[LLD: Elevator](43-lld-elevator/LLD.md)** – State, strategy (scheduling)
- **[LLD: Car Rental](44-lld-car-rental/LLD.md)** – Factory, state (booking)
- **[LLD: Logging System](45-lld-logging/LLD.md)** – Chain of Responsibility, Singleton
- **[LLD: Tic-Tac-Toe](46-lld-tic-tac-toe/LLD.md)** – State, minimal design
- **[LLD: Chess](47-lld-chess/LLD.md)** – State, strategy (piece move)
- **[LLD: File System](48-lld-file-system/LLD.md)** – Composite
- **[LLD: Splitwise](49-lld-splitwise/LLD.md)** – Simplify algorithm
- **[LLD: Vending Machine](50-lld-vending-machine/LLD.md)** – State
- **[LLD: ATM](51-lld-atm/LLD.md)** – State, chain (validation)
- And more in Part 4 table in README.

See main [System Design README](../README.md) for full index.
