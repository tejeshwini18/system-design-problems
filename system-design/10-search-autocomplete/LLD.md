# Low-Level Design: Search Autocomplete System

## 1. APIs

```http
GET    /v1/suggest?q=app&limit=10
  Response: {
    "suggestions": [
      { "query": "apple store", "count": 12000 },
      { "query": "apple music", "count": 8000 }
    ]
  }

POST   /v1/search                      (internal or from search service)
  Body: { "query": "apple store", "user_id": "u123" }
  (Records query for aggregation; returns search results separately)
```

---

## 2. Key Classes / Modules

```text
AutocompleteService
  - getSuggestions(prefix: string, limit: int) → suggestions[]

Trie (in-memory)
  - root: Node
  - insert(query: string, count: int)
  - search(prefix: string, limit: int) → List<(query, count)>

TrieNode
  - children: Map<char, TrieNode>
  - suggestions: List<(query, count)>  // top M at this node, sorted by count

QueryLogger
  - log(query: string, userId?: string, timestamp: long)

AggregationJob
  - run(): aggregate last 7 days → query -> count
  - buildTrie(counts: Map<query, count>) → Trie
  - deployTrie(trie: Trie)  // swap in service
```

---

## 3. Trie Insert (Build) Algorithm

```text
For each (query, count) in aggregated_counts:
  node = root
  for char c in query:
    if c not in node.children: node.children[c] = new TrieNode()
    node = node.children[c]
    node.suggestions.add((query, count))
    sort node.suggestions by count descending
    keep only first M (e.g. 100) in node.suggestions
```

---

## 4. Trie Search Algorithm

```text
search(prefix, limit):
  node = root
  for char c in prefix:
    if c not in node.children: return []
    node = node.children[c]
  return node.suggestions[:limit]
```

---

## 5. Redis-Based Design (Alternative)

- For each prefix that appears in our query set, maintain a sorted set: key = "suggest:" + prefix, member = full query, score = count.
- Example: prefix "app" → ZADD suggest:app 12000 "apple store" 8000 "apple music"
- getSuggestions(prefix, limit): ZREVRANGE suggest:app 0 limit-1 WITHSCORES
- Update: aggregation job computes counts; for each query, for each prefix of query, ZADD to suggest:prefix. Prune: keep only top 100 per prefix (ZREMRANGEBYRANK 0 -101).

---

## 6. Aggregation Pipeline (Batch)

1. Read query log from last 7 days (or from Kafka offset).
2. Group by query string; count.
3. Sort by count descending; optionally filter (min count, blocklist).
4. Build trie: for each query, insert with count at every prefix.
5. Export trie to file or push to Redis; Autocomplete Service loads new data (swap or reload).

---

## 7. API Response and Limits

- limit cap: e.g. 1–20; default 10.
- prefix length: ignore if length < 2 to avoid huge result sets.
- Sanitize prefix (trim, max length) to avoid abuse.

---

## 8. Error Handling

- Empty prefix: return empty or 400.
- Service unavailable (trie not loaded): 503 with retry-after.

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

