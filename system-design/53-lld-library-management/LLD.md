# LLD: Library Management System

## 1. Requirements

- **Books:** title, author, ISBN, copies (multiple copies of same book); **BookItem** = one physical copy (barcode, status: AVAILABLE/ISSUED/RESERVED).
- **Members:** register, get card; can **borrow** (issue) and **return** books; **limit** on number of books (e.g. 5); **fine** for late return (per day).
- **Issue:** Member borrows BookItem; due date = issue date + 14 days (configurable); BookItem status = ISSUED; record in IssuedBook.
- **Return:** Member returns BookItem; check due date; if overdue, calculate fine; BookItem status = AVAILABLE; record return.
- **Reservation:** If no copy available, member can reserve; when returned, first in queue gets notification and can issue within N days.
- **Optional:** Rack assignment, search by title/author, renew.

---

## 2. Core Classes

```text
Book – isbn, title, author; getAvailableCopies() → List<BookItem>
BookItem – barcode, bookId, status (AVAILABLE/ISSUED/RESERVED), rackId?
Member – memberId, name; getCurrentIssuedCount()
IssueRecord – memberId, bookItemId, issueDate, dueDate, returnDate?
Reservation – memberId, bookId, createdAt; queue per book
Fine – memberId, amount, reason (overdue), status (PAID/UNPAID)
LibraryService – issueBook(memberId, bookItemId); returnBook(bookItemId); reserveBook(memberId, bookId); calculateFine(issueRecord); payFine(memberId, amount)
```

---

## 3. Issue Flow

1. Validate member (not blocked, currentIssuedCount < maxAllowed).
2. Get BookItem by barcode; check status == AVAILABLE; if not, return "not available" or suggest reserve.
3. Create IssueRecord(memberId, bookItemId, issueDate, dueDate); set BookItem.status = ISSUED; increment member’s issued count.
4. If this book had reservations, remove first reservation (and optionally notify that member).

---

## 4. Return Flow

1. Get BookItem; find IssueRecord(memberId, bookItemId) where returnDate is null.
2. returnDate = now; if returnDate > dueDate, fine = (returnDate - dueDate).days * perDayFine; create Fine record; optionally block member until fine paid.
3. Set BookItem.status = AVAILABLE; set record.returnDate; decrement member’s issued count.
4. If there is a reservation for this bookId, assign BookItem to next in queue (or notify them to come and issue).

---

## 5. Reservation

- **Reserve:** If no BookItem available for bookId, add Reservation(memberId, bookId); queue by createdAt.
- **On return:** When a copy of bookId is returned, peek reservation queue; notify first member (or auto-issue if policy); give time window to issue else skip to next.

---

## 6. Key Methods

```text
LibraryService.issueBook(memberId, bookItemBarcode) → IssueRecord | error
LibraryService.returnBook(bookItemBarcode) → fine amount if any
LibraryService.reserveBook(memberId, bookId)
LibraryService.calculateFine(issueRecord) → amount
LibraryService.searchBooks(query) → List<Book>
```

---

## 7. Design Patterns

- **State:** BookItem status (Available, Issued, Reserved); transitions on issue, return, reserve.
- **Observer:** Notify reserved members when book returned.
- **Strategy:** Fine calculation (flat per day vs graduated).

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

