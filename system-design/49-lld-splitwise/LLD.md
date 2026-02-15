# LLD: Splitwise & Simplify Algorithm

## 1. Requirements

- **Users** in a **group**; **expenses:** "A paid 100 for all" or "A paid 50 for B and C."
- **Split types:** Equal, percentage, or exact amount per user.
- **Balances:** Per user: net amount owed (positive) or to receive (negative); simplify so minimum transactions to settle.
- **Simplify (optimal balancing):** Given balances (who owes whom how much), find minimal set of transactions (e.g. A→B: 50, C→B: 30) so everyone is settled.
- **Optional:** Groups, history, categories, currencies.

---

## 2. Core Classes

```text
User – id, name
Group – id, members: List<User>
Expense – id, paidBy: User, amount, splitType (EQUAL/PERCENTAGE/EXACT), splitDetails: Map<User, amount or percent>
  participants: List<User>   // who is part of this expense
ExpenseService – addExpense(paidBy, amount, participants, splitType, details), getBalance(user), getBalancesInGroup(group)
Balance – userId, amount (positive = owes, negative = owed)
Settlement – fromUser, toUser, amount
SimplifyService – simplify(balances: Map<User, amount>) → List<Settlement>
```

---

## 3. Balance Calculation

- For each expense: paidBy credits (lent) amount; each participant debits their share.
- **Net balance per user:** Sum of (amount they paid) − (their share in all expenses). Positive = they are owed; negative = they owe.
- **Per group:** Filter expenses by group; compute net balance for each member.

---

## 4. Simplify Algorithm (Minimum Transactions)

**Input:** Map<User, balance> where balance = net (positive = owed money, negative = owes money). Sum of balances = 0.

**Idea:** Treat as flow. Greedy: repeatedly pick one with max debt (owes) and one with max credit (owed); settle min of the two; update; repeat.

**Steps:**
1. **Max heap (debtors):** users with balance < 0 (they owe). **Min heap (creditors):** users with balance > 0 (they are owed). Or two lists sorted by balance.
2. While both non-empty: debtor = max owe (e.g. −50); creditor = max owed (e.g. +30). amount = min(50, 30) = 30. Settlement: debtor → creditor: 30. Update: debtor balance += 30 → −20; creditor balance −= 30 → 0. Remove creditor; push debtor back if balance != 0.
3. Result: List of (from, to, amount).

**Optimality:** This gives minimum number of transactions (at most n−1 for n users) and minimizes total flow in the sense of clearing all balances.

**Alternative (graph):** Model as directed graph (A owes B 10 → edge A→B weight 10). Simplify by path compression: if A→B 10 and B→C 10, then A→C 10 and remove B from chain. More complex for full optimality (min transactions); greedy above is standard interview solution.

---

## 5. Key Methods

```text
ExpenseService.addExpense(paidBy, amount, participants, splitType, splitDetails)
ExpenseService.getBalancesInGroup(groupId) → Map<UserId, balance>
SimplifyService.simplify(balances) → List<Settlement>
```

---

## 6. Design Patterns

- **Strategy:** SplitStrategy (EqualSplit, PercentageSplit, ExactSplit) for different split types.
- **Observer:** Notify users when new expense added or settlement suggested.
