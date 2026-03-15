# Low-Level Design: Metrics & Analytics Platform

## 1. APIs

### Ingest
```http
POST   /v1/track
  Body: {
    "events": [
      { "event": "signup", "user_id": "u123", "properties": { "country": "US", "source": "web" }, "timestamp": "ISO8601" }
    ]
  }
  Response: 202 Accepted

POST   /v1/metrics
  Body: { "name": "api_latency_ms", "labels": {"path":"/api"}, "value": 120, "timestamp": 1234567890 }
```

### Query
```http
GET    /v1/analytics/events/count?event=signup&from=...&to=...&group_by=country
  Response: { "data": [ { "country": "US", "count": 1000 }, ... ] }

GET    /v1/analytics/funnel?steps=signup,pay,subscribe&from=...&to=...
  Response: { "steps": [ { "name": "signup", "count": 10000 }, { "name": "pay", "count": 2000 }, ... ] }

POST   /v1/query
  Body: { "query": "SELECT event, count(*) FROM events WHERE date = '2024-02-14' GROUP BY event" }
  Response: { "columns": [...], "rows": [...] }
```

### Dashboards
```http
GET    /v1/dashboards
GET    /v1/dashboards/:id
POST   /v1/dashboards
PATCH  /v1/dashboards/:id
GET    /v1/dashboards/:id/data?from=...&to=...   // returns chart data
```

---

## 2. Database / Store Schemas

**Events (Kafka topic):** key = tenant_id or (user_id, timestamp); value = JSON event.

**Pre-aggregates (time-series or OLAP):**
```sql
event_counts (date DATE, event_name VARCHAR, dimension_key VARCHAR, dimension_value VARCHAR, count BIGINT, PRIMARY KEY (date, event_name, dimension_key, dimension_value));
-- or in TSDB: metric event_signup_count{country=US} 1000 @timestamp
```

**Dashboards:**
```sql
dashboards (id PK, name VARCHAR, created_at);
dashboard_charts (id PK, dashboard_id FK, title VARCHAR, metric VARCHAR, filters JSON, group_by VARCHAR[], viz_type VARCHAR, sort_order INT);
```

---

## 3. Key Classes / Modules

```text
IngestService
  - track(events[])  // validate, produce to Kafka

StreamProcessor
  - consume(): for each event, aggregate by (date, event_name, dimensions) and flush to TSDB/OLAP
  - flushAggregates()  // every 1 min or 10k events

QueryService
  - getEventCount(eventName, from, to, groupBy) → series or breakdown
  - getFunnel(steps[], from, to) → step counts
  - runSql(query) → run against data lake / OLAP

DashboardService
  - getDashboard(id) → dashboard with chart configs
  - getDashboardData(id, from, to) → for each chart, QueryService query; return combined data

AggregationStore (TSDB or OLAP)
  - increment(metric, labels, timestamp_bucket, delta)
  - query(metric, from, to, groupBy) → [(labels, timestamp, value)]
```

---

## 4. Pre-Aggregation (Stream Processor)

- Maintain in-memory map: key = (date, event_name, country, device), value = count.
- On event: parse event_name and dimensions from properties; key = (date(event.timestamp), event_name, properties.country, properties.device); map[key]++.
- Every 60s or 10K events: for each key, write to TSDB (e.g. INCR or batch insert to OLAP); clear map.
- Downsampling: for 1h/1d rollups, separate job reads 1m data and writes 1h/1d buckets.

---

## 5. Funnel Query (Steps)

- Funnel: signup → pay → subscribe in date range.
- For each user, compute first timestamp of each event in steps (from raw events or pre-aggregated user-level table).
- Count users with step1; count users with step1 and step2 (and step2.ts > step1.ts); etc.
- Implementation: SQL with window functions or scan events and build user-level funnel state; then count. For scale, use pre-built funnel table (user_id, step_1_ts, step_2_ts, ...) updated by stream processor.

---

## 6. Dashboard Data (Steps)

1. Load dashboard and chart configs.
2. For each chart: build query (metric, filters, group_by, from, to); call QueryService.getEventCount or getMetricSeries.
3. Combine results into response: { charts: [ { chartId, data: [...] } ] }.
4. Cache by (dashboard_id, from, to) with TTL 1–5 min.

---

## 7. SQL on Data Lake

- Events stored as Parquet: date, event_name, user_id, properties (JSON or flattened columns), timestamp.
- Query: "SELECT event_name, count(*) FROM events WHERE date BETWEEN ? AND ? GROUP BY event_name" → run via Presto/Athena/Spark; return result set.
- Limit: max scan size, timeout (e.g. 5 min); queue long-running queries.

---

## 8. Error Handling

- Ingest: invalid event (missing required field) → drop or send to DLQ; return 207 with per-event status.
- Query timeout: return 504; suggest narrowing time range.
- Rate limit: 429 on ingest and heavy query endpoints.
