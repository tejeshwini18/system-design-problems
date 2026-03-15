# Low-Level Design: Database Sharding

## 1. Routing Interface

```text
ShardRouter.getShard(shardKey: string) → ShardConnection
ShardRouter.getShardForRead(shardKey: string) → ShardConnection   // optional: replica
```

- **ShardKey:** From entity (e.g. user_id, order_id). For composite, e.g. (user_id, order_id) use user_id for orders table so all orders of a user are on same shard.

---

## 2. Key Classes / Modules

```text
ShardRouter
  - numShards: int
  - hash(key) → shardId: int
  - getShard(key) → connection   // connectionPool[shardId]
  - getShardById(shardId) → connection

HashStrategy
  - hash(key) → int   // CRC32 or MurmurHash
  - shardId = abs(hash(key)) % numShards

DirectoryStrategy (alternative)
  - lookup(key) → shardId   // from DB or cache
  - updateMapping(key, shardId)   // for rebalance

OrderRepository
  - create(order): shardKey = order.userId; shard = router.getShard(shardKey); shard.insert("orders", order)
  - getById(orderId): need userId → use order_id_to_user_id cache/table to get userId then route; or scatter all shards (SELECT WHERE order_id = ?)
  - getByUserId(userId, limit): shard = router.getShard(userId); shard.query("SELECT * FROM orders WHERE user_id = ? ORDER BY created_at DESC LIMIT ?", userId, limit)
```

---

## 3. Hash Function (Consistent Distribution)

```text
hash(key):
  h = murmur3_or_crc32(key)
  return abs(h) % numShards
```

- **Consistent hashing:** If using for rebalance, use ring with virtual nodes; shardId = findNextNodeOnRing(hash(key)).

---

## 4. Secondary Index (Lookup by Non-Shard Key)

- **Option A:** Global index table (e.g. order_id → user_id) stored in separate store (Redis, or dedicated “index” DB); lookup order_id → user_id, then route by user_id to get full row.
- **Option B:** Scatter-gather: query every shard SELECT * FROM orders WHERE order_id = ?; return first non-null (only one shard has it).
- **Option C:** Index table co-located: each shard has (order_id, user_id) for orders in that shard; query all shards for order_id; get user_id; then get full order from that shard (or return from scatter if you already fetched full row in scatter).

---

## 5. Rebalance (Directory Approach)

- **Directory:** key_range or key → shard_id; stored in DB or ZooKeeper; cached in router.
- **Add shard:** Create new shard; for keys that should move, update directory (key → new_shard_id); migrate data (copy then delete); dual-write during transition; switch reads then writes.
- **Router:** getShard(key) → directory.lookup(key) or hash(key) if no directory.

---

## 6. Schema (Per Shard)

- **Same DDL on every shard:** orders (id, user_id, amount, created_at); primary key id; index (user_id, created_at).
- **Insert:** id can be UUID or snowflake; shard_key = user_id; insert into shard determined by user_id.
- **No foreign key across shards:** e.g. orders.user_id references users.id but users may be on different shard; enforce in application or use global users table (replicated).

---

## 7. Error Handling

- **Shard down:** Router returns connection failure; application retries or returns 503 for that shard’s data.
- **Rebalance:** During move, read from old; write to both (dual-write); then switch to new; then stop writing to old and remove old data.
- **Cross-shard transaction:** Not supported natively; use saga (compensating actions) or accept eventual consistency.

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

