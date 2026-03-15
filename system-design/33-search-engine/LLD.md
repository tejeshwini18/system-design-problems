# Low-Level Design: Search Engine

## 1. APIs (User-Facing)

```http
GET    /search?q=apple+pie&limit=20&offset=0
  Response: { "results": [ { "title", "url", "snippet", "score" } ], "total": 1000 }
```

---

## 2. Key Classes / Modules

```text
UrlFrontier
  - push(url, priority)
  - pop() → url   // by priority; respect per-domain delay
  - dedupe(url) → boolean   // already seen?

Crawler
  - fetch(url) → html
  - obeyRobots(url), rateLimit(domain)
  - run(): while url = frontier.pop(); page = fetch(url); parser.parse(page); extract links; frontier.push(links)

Parser
  - parse(html) → { title, text, links[] }
  - extractText(dom), normalize(text) → tokens

Indexer
  - addDocument(docId, url, title, tokens)
  - for each term in tokens: append to posting list (docId, tf, positions)
  - posting lists stored per term (or shard by term)

Index (inverted)
  - getPostingList(term) → [(docId, tf, positions), ...]
  - intersect(list1, list2) → list   // for AND query

Ranker
  - score(docId, terms, query) → score   // BM25 + pagerank
  - BM25(tf, df, N, docLen, avgLen), IDF(N, df)

QueryService
  - search(queryStr, limit) → results[]
  - parseQuery → terms[]; lookup each term; merge; rank; topK; fetch snippets
```

---

## 3. URL Frontier (Politeness)

- **Per-domain queue:** Map<domain, Queue<url>>; global priority queue of (next_fetch_time, domain). Pop domain whose next_fetch_time <= now; pop URL from that domain; fetch; set next_fetch_time = now + delay (e.g. 1s).
- **Dedupe:** Bloom filter or set of seen URLs (normalized); before push, check and skip if seen.

---

## 4. Posting List Merge (AND Query)

- Terms: ["apple", "pie"]. Get list1 = posting("apple"), list2 = posting("pie").
- **Intersect:** Two-pointer: advance pointer in list with smaller doc_id until equal; add doc_id to result; advance both. For K terms: pairwise merge or use smallest list first and check others.
- **Phrase:** After doc_id intersection, check positions: for "apple pie", positions of "apple" and "pie" must be consecutive in doc.

---

## 5. BM25 (Simplified)

```text
IDF(term) = log((N - df + 0.5) / (df + 0.5) + 1)
score(doc, term) = IDF(term) * (tf * (k1+1)) / (tf + k1 * (1 - b + b * docLen/avgLen))
total_score(doc, query) = sum over terms of score(doc, term)
```
- N = total docs; df = doc frequency of term; tf = term freq in doc; docLen, avgLen; k1, b constants.

---

## 6. Snippet Generation

- Store first 500 chars of main text per doc (or full text and extract at query time).
- For each matching term, find position in text; extract surrounding window (e.g. 80 chars); highlight term; return as snippet. If multiple terms, prefer span that contains most terms.

---

## 7. Error Handling

- **Crawler:** 4xx/5xx → skip or retry with backoff; timeout → retry; robots.txt disallow → skip URL.
- **Query:** Term not in index → empty list for that term; merge yields empty → return no results.
- **Index shard down:** Return partial results or 503 for that shard.

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

