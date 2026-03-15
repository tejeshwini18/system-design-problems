# Low-Level Design: How Bluesky Works (LLD)

## 1. APIs (Representative)

### Identity
```http
GET    /.well-known/atproto-did?handle=alice.bsky.social   → DID
GET    https://<PDS>/xrpc/com.atproto.identity.resolveHandle?handle=alice.bsky.social  → DID, doc
```

### Repository (PDS)
```http
GET    /xrpc/com.atproto.repo.getRecord?repo=<DID>&collection=app.bsky.feed.post&rkey=<rkey>
GET    /xrpc/com.atproto.repo.listRecords?repo=<DID>&collection=app.bsky.feed.post&limit=50
POST   /xrpc/com.atproto.repo.createRecord   Body: { repo, collection, record, rkey? }
POST   /xrpc/com.atproto.repo.deleteRecord   Body: { repo, collection, rkey }
```

### Feed (App View)
```http
GET    /xrpc/app.bsky.feed.getTimeline?feed=...&limit=50&cursor=...
GET    /xrpc/app.bsky.feed.getAuthorFeed?actor=<DID>&limit=50
```

---

## 2. Record Schemas (Conceptual)

**Post:** `{ "text": "...", "createdAt": "ISO8601", "reply": { "parent": { "uri": "at://...", "cid": "..." }, "root": { ... } } }`  
**Profile:** `{ "displayName": "...", "description": "...", "avatar": { "ref": "..." } }`  
**Follow:** `{ "subject": "did:plc:..." }`  
**Like:** `{ "subject": { "uri": "at://did/feed.post/rkey", "cid": "..." } }`

- **Collection** = namespace: e.g. `app.bsky.feed.post`, `app.bsky.actor.profile`.
- **rkey** = unique key within collection (e.g. TID for posts).

---

## 3. Commit Structure (Repository)

- **Commit** = { prev: CID | null, ops: [ { action: "create"|"update"|"delete", path: "collection/rkey", record? } ], sig }.
- **path** = collection + rkey (e.g. app.bsky.feed.post/3k2abc).
- PDS appends commit to user’s log; state is derived by applying ops in order (last write wins per path).

---

## 4. Key Classes / Modules

```text
IdentityResolver
  - resolveHandle(handle) → did, didDoc
  - getPDSUrl(did) → url  // from didDoc service endpoint

PDS
  - getRecord(repoDid, collection, rkey) → record
  - listRecords(repoDid, collection, limit, cursor) → records[]
  - createRecord(repoDid, collection, record, rkey?) → rkey  // append commit
  - deleteRecord(repoDid, collection, rkey)

FeedGenerator
  - getTimeline(userDid, limit, cursor) → feed items, nextCursor
  - getFollows(userDid) → list of DIDs  // from user's repo, collection follows
  - fetchRecentPosts(pdsUrl, repoDid, limit) → posts  // from PDS listRecords
  - mergeAndRank(postsFromManyRepos) → ordered feed
```

---

## 5. Feed Generation (Steps)

1. getFollows(userDid): from user’s PDS, list records in app.bsky.graph.follow → list of subject DIDs.
2. For each followed DID: resolve PDS URL (cache); call listRecords or getRecent for app.bsky.feed.post; collect recent posts with (uri, createdAt, author).
3. Merge all posts; sort by createdAt (or rank by engagement); apply cursor-based pagination.
4. Optionally hydrate: for each post ref, fetch full record from PDS if not cached.
5. Return feed items (post ref + snippet) and nextCursor.

---

## 6. Handle and DID Resolution

- **Handle** = domain-like (alice.bsky.social). DNS: _atproto.<handle> TXT or well-known URL returns DID.
- **DID document** (fetched from DID method, e.g. PLC): contains service endpoint for PDS (and signing keys). Cache DID doc and PDS URL with TTL.

---

## 7. Error Handling

- **PDS down:** Feed generator skips that user’s posts for this request; retry later; optional fallback cache.
- **Invalid commit:** PDS rejects (400); client retries or shows error.
- **Migration:** User points DID document to new PDS; clients and feed generators re-resolve; old PDS may redirect or serve read-only until migration complete.

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

