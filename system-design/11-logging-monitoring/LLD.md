# Low-Level Design: Logging & Monitoring System

## 1. APIs

### Log Ingest (agent or app)
```http
POST   /v1/logs
  Body: { "logs": [ { "timestamp": "ISO8601", "level": "info", "message": "...", "service": "api", "host": "i-123", "fields": {} } ] }
  Headers: Authorization: Bearer <api_key>

POST   /v1/metrics
  Body: { "metrics": [ { "name": "http_requests_total", "labels": {"method":"GET","path":"/api"}, "value": 1, "timestamp": 1234567890 } ] }
```

### Search & Query
```http
GET    /v1/logs/search?q=error&service=api&from=...&to=...&size=20
  Response: { "hits": [ { "timestamp", "message", "level", "service", ... } ], "total": N }

GET    /v1/metrics/query?query=rate(http_requests_total[5m])&from=...&to=...
  Response: { "data": [ { "metric": {...}, "values": [[ts, v], ...] } ] }
```

### Alerts
```http
GET    /v1/alerts                 List active alerts
GET    /v1/alert-rules            List rules
POST   /v1/alert-rules            Create rule (expression, for, severity, notify)
```

---

## 2. Database / Store Schemas

**Logs (Elasticsearch index mapping):**
```json
{
  "mappings": {
    "properties": {
      "timestamp": { "type": "date" },
      "level": { "type": "keyword" },
      "message": { "type": "text" },
      "service": { "type": "keyword" },
      "host": { "type": "keyword" },
      "trace_id": { "type": "keyword" },
      "fields": { "type": "object", "enabled": false }
    }
  }
}
```
Index naming: logs-YYYY.MM.DD (roll daily).

**Metrics (time-series):**  
(Conceptual; actual schema depends on TSDB, e.g. Prometheus.)
- Series: (metric_name, label1=val1, ...) → list of (timestamp, value).
- Downsampled tables: same schema with 1h/1d resolution.

**Alert rules:**
```sql
alert_rules (id PK, name, expression TEXT, for_seconds INT, severity, notify_channel, enabled BOOLEAN, created_at)
```

---

## 3. Key Classes / Modules

```text
LogIngestService
  - ingest(batch: LogBatch)  // validate, parse, bulk index to ES

LogSearchService
  - search(query, filters, from, to, size) → hits  // build ES query, execute

MetricsIngestService
  - ingest(batch: MetricBatch)  // write to TSDB

MetricsQueryService
  - query(expression, from, to) → series  // parse PromQL-like, execute

AlertEvaluator
  - evaluateRules()  // periodic: for each rule, run expression; if true for 'for' duration, fire alert
  - notify(alert, channel)

LogConsumer (from Kafka)
  - consume(): for each batch, LogIngestService.ingest(batch)
```

---

## 4. Log Search Query (Elasticsearch)

- q = "error" → message full-text match.
- filters: service=api, level=error, range timestamp [from, to].
- Build bool query: must match message, filter by service/level/range.
- Sort by timestamp desc; size = 20; optionally highlight.

---

## 5. Alert Evaluation (Steps)

1. Cron every 1m (or 30s): load all enabled rules.
2. For each rule: run metrics query (e.g. rate(errors[5m]) > 0.05).
3. If result true: check "for" (e.g. 5m); if true for 5 consecutive evaluations, transition to firing; send notification once (or on resolve).
4. Store alert state (pending/firing/resolved) in DB or Redis; dedupe notifications.

---

## 6. Retention and Cold Storage

- Hot: last 30 days in Elasticsearch; delete index logs-* older than 30d (or curator job).
- Cold: before delete, snapshot indices to S3 (optional); restore on demand for search.
- Metrics: raw 15s for 7d; 1m for 30d; 1h for 1y (downsampling job).

---

## 7. Error Handling

- Ingest: invalid JSON or missing required field → drop or send to dead-letter; return 207 with per-item status.
- Search timeout: return partial results and timeout flag.
- Alert notification failure: retry with backoff; mark channel unhealthy after N failures.

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

