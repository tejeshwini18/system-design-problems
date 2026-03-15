# Low-Level Design: Web Crawler

## 1. APIs (Control Plane)

```http
POST   /v1/crawl/start             Start crawl (seed_urls[], max_urls?, politeness_delay?)
GET    /v1/crawl/status             Status (queued, fetched, failed, rate)
POST   /v1/crawl/stop               Stop crawl
GET    /v1/crawl/urls?status=seen   List URLs (optional export)
```

- Crawler itself is typically a long-running job; API for control and monitoring.

---

## 2. Key Classes / Modules

```text
UrlFrontier
  - add(url, priority?)
  - next() → url | null   // respect per-domain delay
  - perDomain: Map<domain, Queue<url>>
  - nextFetchTime: Map<domain, timestamp>
  - getNextDomain() → domain where nextFetchTime <= now

UrlNormalizer
  - normalize(url) → canonical string
  - lowercase host, decode path, remove fragment, sort query, remove default port

Deduplicator
  - seen(url) → boolean
  - add(url)
  - Bloom filter + DB/cache for persistence

RobotsChecker
  - canFetch(url) → boolean
  - getOrFetchRobots(host) → rules; cache
  - check(path, rules) → allow/disallow

Fetcher
  - fetch(url) → Response { status, body, headers }
  - timeout, redirect limit, User-Agent

Parser
  - parse(html, baseUrl) → { links: url[], title?, text? }
  - extractLinks(dom, baseUrl); normalize each link
```

---

## 3. Scheduler Loop (Next URL)

```text
while true:
  domain = getNextDomain()   // domain with nextFetchTime <= now and non-empty queue
  if domain is null: sleep(1); continue
  url = perDomain[domain].pop()
  if url is null: continue
  nextFetchTime[domain] = now + delay
  return url
```

---

## 4. Worker Loop

```text
while url = frontier.next():
  if not robotsChecker.canFetch(url): continue
  response = fetcher.fetch(url)
  if response.status != 200: log; continue
  doc = parser.parse(response.body, url)
  for link in doc.links:
    normalized = normalizer.normalize(link)
    if not deduplicator.seen(normalized):
      deduplicator.add(normalized)
      frontier.add(normalized)
  optional: documentStore.save(url, response.body)
```

---

## 5. URL Normalization (Steps)

1. Parse URL; default scheme to https.
2. Lowercase host; strip trailing dot; strip default port (80, 443).
3. Path: decode %xx; remove default /; collapse repeated /; optional: sort path segments?
4. Remove fragment (#...).
5. Query: optional sort key=value; remove utm_* if desired.
6. Rebuild string; use as key.

---

## 6. Robots.txt Parsing

- Split by lines; look for "User-agent: *" block; then "Disallow: /path" or "Allow: /path"; "Crawl-delay: 5".
- For URL path, check each rule (Disallow/Allow); first match wins or longest prefix; if Disallow matches, canFetch = false.
- Cache key = host; value = parsed rules; TTL 24h.

---

## 7. Error Handling

- **Timeout / connection error:** Retry up to 3 times with backoff; then mark URL failed and skip.
- **4xx/5xx:** Log; skip URL (or re-queue with lower priority for 5xx).
- **Parse error:** Skip links for that page; still mark URL as crawled to avoid retry loop.
- **Robots fetch fail:** Assume allow (fail open) or disallow (fail closed) per policy.

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

