# LLD: ATM System

## 1. Requirements

- **ATM** has: card reader, keypad, screen, cash dispenser, printer (receipt).
- **User:** Insert card → enter PIN → choose operation (balance, withdraw, deposit, transfer) → complete.
- **Card:** cardNumber, pin (hashed); linked to **Account(s)**.
- **Account:** accountId, balance; debit/credit with validation.
- **Withdraw:** Check balance ≥ amount; dispense cash; update balance; print receipt.
- **Deposit:** Accept cash (or check); update balance; print receipt.
- **Transfer:** From one account to another (same or different user); validate balance; debit one, credit other.
- **Session:** One session per card; timeout after inactivity; eject card on logout or timeout.

---

## 2. Core Classes

```text
Card – cardNumber, pinHash, accountIds[]
Account – accountId, balance; debit(amount), credit(amount), getBalance()
BankService (or CardValidator) – validateCard(cardNumber, pin) → Account[] | null
ATM – currentCard, currentAccount, state (Idle/InsertCard/Pin/Operation/...)
ATMState (State pattern) – IdleState, PinEntryState, OperationState, etc.
CashDispenser – dispense(amount); canDispense(amount)
DepositSlot – acceptCash() (simplified: just credit)
Printer – printReceipt(message)
```

---

## 3. Flow (Withdraw)

1. **Idle** → user inserts card → **CardInserted**; read cardNumber.
2. **PinEntry** → user enters PIN; BankService.validate(cardNumber, pin) → accounts; if fail, retry or block.
3. **SelectAccount** → user picks account → currentAccount set.
4. **OperationMenu** → user selects Withdraw → **WithdrawState**.
5. User enters amount; validate amount > 0 and amount ≤ currentAccount.getBalance() and CashDispenser.canDispense(amount).
6. currentAccount.debit(amount); CashDispenser.dispense(amount); Printer.printReceipt(...); return to OperationMenu.
7. User selects Exit → eject card; **Idle**.

---

## 4. State Diagram (Simplified)

```
Idle → (insert card) → PinEntry → (valid PIN) → SelectAccount → OperationMenu
  ↑                                                                     |
  |                                                                     |
  └──────────────────────── (eject / timeout) ←────────────────────────┘
```

---

## 5. Concurrency & Safety

- **One session per ATM:** Mutex so only one user at a time (or model as single-threaded).
- **Balance update:** BankService (remote) handles debit/credit with transaction; ATM calls API; on failure, don’t dispense.
- **PIN:** Never store plain PIN; validate via hash; limit retries (e.g. 3) then block card.

---

## 6. Design Patterns

- **State:** ATMState and subclasses (IdleState, PinEntryState, WithdrawState, etc.); handle user actions per state.
- **Chain of Responsibility:** Validate amount (positive → ≤ balance → dispenser capacity) in chain.
- **Facade:** ATM class as facade over CardReader, Keypad, Screen, CashDispenser, BankService.

---

## 7. Key Methods

```text
ATM.insertCard(cardNumber)
ATM.enterPin(pin)
ATM.selectAccount(accountId)
ATM.withdraw(amount) → success/failure
ATM.deposit(amount)
ATM.transfer(toAccountId, amount)
ATM.ejectCard()
ATM.getBalance()
```
