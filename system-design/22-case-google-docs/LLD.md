# Low-Level Design: Google Docs–Style Collaboration

## 1. APIs

```http
GET    /v1/documents/:id                 Load document (snapshot + optional ops after version)
POST   /v1/documents                    Create document (title) → doc_id
PUT    /v1/documents/:id                 Save snapshot (optional; server can do periodically)
```

**WebSocket:** `WS /v1/docs/:doc_id?token=...`

**Client → Server:**
- `{ "type": "op", "op": { "kind": "insert", "index": 5, "text": "X" }, "client_id": "...", "version": 3 }`
- `{ "type": "cursor", "position": 10, "selection": { "start": 10, "end": 12 } }`
- `{ "type": "ping" }`

**Server → Client:**
- `{ "type": "op", "op": {...}, "version": 4 }`  // transformed op to apply
- `{ "type": "cursor", "user_id": "...", "position": 10, "selection": {...} }`
- `{ "type": "presence", "users": [ { "user_id": "...", "cursor": ... } ] }`
- `{ "type": "pong" }`

---

## 2. Document Model (Simplified)

- **Document:** List of characters or “blocks” (paragraph, heading) with formatting.
- **Op:** Insert(index, text), Delete(index, length), or Format(index, length, attributes).
- **Version:** Monotonically increasing; server assigns after each applied op.
- **Client:** Tracks local version; sends op with “after version”; server rejects if version mismatch (client must reconcile).

---

## 3. Key Classes / Modules

```text
CollaborationService
  - applyOp(docId, clientId, op, clientVersion) → transformedOp, newVersion
  - getDocument(docId) → snapshot, version
  - broadcast(docId, message)  // to all clients in doc

DocumentStore
  - getSnapshot(docId) → (content, version)
  - appendOp(docId, op, version) → newVersion
  - getOpsAfter(docId, version) → ops[]  // for catch-up

OTEngine (or CRDT merge)
  - transform(op1, op2) → (op1', op2')  // so apply(op1); apply(op2') === apply(op2); apply(op1')
  - apply(state, op) → newState

PresenceManager
  - setCursor(docId, userId, cursor)
  - getPresence(docId) → list of (userId, cursor)
  - remove(docId, userId)  // on disconnect
```

---

## 4. Apply Op (Server) – Steps

1. Load document state and current version for doc_id.
2. If client_version != server version: fetch ops from client_version to server version; transform incoming op through those ops (OT); result is op' that applies to current server state.
3. Apply op' to server state; increment version; persist op' to op log (and optionally update snapshot).
4. Broadcast { type: "op", op: op', version: newVersion } to all other clients in doc.
5. Return newVersion to sender (ack).

---

## 5. Client-Side Apply

- On user input: create op; send to server; optionally apply optimistically (if no concurrent edit in same region).
- On receive op from server: apply to local state; update version; re-render. If optimistic apply was wrong, reconcile: either revert and re-apply server ops, or transform local pending ops against server op and re-apply.

---

## 6. OT Transform Example (Two Inserts)

- State: "ab"; Client A: Insert(1, "x") → "axb"; Client B: Insert(2, "y") → "ayb".
- Server receives A first: state = "axb", version 2. B’s op Insert(2, "y") must be transformed: after A’s insert, index 2 is after "x", so B' = Insert(3, "y") → "axyb". Broadcast B' to A; A applies Insert(3, "y") → "axyb". Same final state.

---

## 7. Persistence Schema (Conceptual)

```sql
documents (doc_id PK, title, snapshot JSON or TEXT, version INT, updated_at);
document_ops (doc_id FK, version INT, op JSON, author_id, created_at, PRIMARY KEY (doc_id, version));
```

- Snapshot refreshed every K ops or every T seconds; ops kept for replay and history.

---

## 8. Error Handling

- **Version conflict:** Client sent version 5, server at 7 → return 409 with current snapshot or ops 5..7; client replays and re-sends its op.
- **Disconnect:** Client reconnects; load snapshot + get ops after snapshot version; replay; resend any pending local ops with correct version.
- **Large doc:** Snapshot can be chunked or lazy-loaded by block id for very large docs.

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

