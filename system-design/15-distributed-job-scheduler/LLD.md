# Low-Level Design: Distributed Job Scheduler

## 1. APIs

```http
POST   /v1/jobs
  Body: { "type": "send_report", "cron": "0 9 * * *", "params": { "report_id": "r1" } }
  Response: { "job_id": "...", "next_run_at": "..." }

POST   /v1/jobs/run-once
  Body: { "type": "sync", "run_at": "2024-02-15T10:00:00Z", "params": {} }
  Response: { "job_id": "...", "run_id": "..." }

GET    /v1/jobs/:id
GET    /v1/jobs/:id/executions?limit=20
DELETE /v1/jobs/:id
POST   /v1/jobs/:id/pause
POST   /v1/jobs/:id/resume
```

---

## 2. Database Schemas

```sql
jobs (
  job_id PK, type VARCHAR, cron VARCHAR NULL, run_at TIMESTAMP NULL,
  params JSON, next_run_at TIMESTAMP NULL, enabled BOOLEAN DEFAULT true,
  created_at, updated_at
);
CREATE INDEX idx_next_run ON jobs(next_run_at) WHERE enabled = true;

executions (
  execution_id PK, job_id FK, run_id VARCHAR UNIQUE, status ENUM(pending, running, success, failed),
  started_at, finished_at, output TEXT, error TEXT, worker_id VARCHAR
);
CREATE INDEX idx_job_run ON executions(job_id, run_id);
```

---

## 3. Key Classes / Modules

```text
JobService
  - createJob(type, cron, params) → jobId
  - createOneOff(type, runAt, params) → jobId
  - getJob(jobId) → job
  - listExecutions(jobId, limit) → executions[]
  - pause(jobId) / resume(jobId)

SchedulerLoop
  - run(): every 1 min, getDueJobs() → for each: tryEnqueue(job)
  - getDueJobs(): SELECT * FROM jobs WHERE enabled AND next_run_at <= NOW() + 1min
  - tryEnqueue(job): lock(job.id, job.next_run_at) → if ok: publish to queue; update next_run_at; unlock

CronEvaluator
  - nextRunAfter(cron, fromTime) → nextTimestamp  // e.g. parse "0 9 * * *" and return next 9:00

Worker
  - poll(): get message from queue (job_id, run_id, type, params)
  - process(msg): tryLock(run_id); if ok: execute(type, params); record execution; ack
  - execute(type, params): dispatch to handler registry[type]

LockService (Redis)
  - tryLock(key, owner, ttlSeconds) → boolean
  - unlock(key, owner)
```

---

## 4. Scheduler Enqueue (Steps)

1. dueJobs = SELECT * FROM jobs WHERE enabled AND next_run_at <= NOW() + 60s ORDER BY next_run_at LIMIT 1000.
2. For each job: key = "sched:" + job_id + ":" + next_run_at (e.g. epoch).
3. Redis SET key owner NX EX 120. If not ok, skip (another scheduler got it).
4. Publish to Kafka/SQS: { job_id, run_id: next_run_at, type, params }.
5. next = CronEvaluator.nextRunAfter(job.cron, next_run_at). UPDATE jobs SET next_run_at = next WHERE job_id = ?.
6. Redis DEL key (or let TTL 120s).

---

## 5. Worker Execute (Steps)

1. Receive message; run_id = message.run_id.
2. Redis SET "exec:" + run_id worker_id NX EX 3600. If not ok, ack message (idempotent: already running or done).
3. INSERT executions (job_id, run_id, status=running, started_at, worker_id).
4. handler = registry[message.type]; handler.run(message.params).
5. On success: UPDATE executions SET status=success, finished_at; ack message. On exception: UPDATE status=failed, error=...; retry (requeue) or send to DLQ after N retries.
6. Redis DEL "exec:" + run_id.

---

## 6. Cron Parsing (Simplified)

- Format: minute hour day month weekday (e.g. "0 9 * * *" = daily 9:00).
- nextRunAfter(cron, from): iterate from from+1min in 1-min steps; check if (minute, hour, day, month, weekday) matches cron; return first match. Use library (e.g. cron-parser) in production.

---

## 7. Retry Policy

- On handler exception: requeue with visibility timeout (e.g. 60s, 300s, 900s) for 3 attempts; then move to DLQ.
- Or: update execution status failed; separate retry job that re-enqueues by run_id with backoff.

---

## 8. Error Handling

- Duplicate run_id: worker skips (already completed or running); ack to avoid poison message.
- Job disabled: scheduler skips when loading due jobs (enabled = true).
- Worker crash: execution lock TTL expires; message becomes visible again; another worker may pick up (idempotent handler or check execution status before run).
