# LLD: LMS & Calendar (Summary)

## LMS Key APIs

```http
GET/POST   /v1/courses   List, create
GET/POST   /v1/courses/:id/lessons
POST       /v1/courses/:id/enroll   userId
PUT        /v1/progress   lessonId, userId, completed
POST       /v1/assignments   Create; submit(assignmentId, userId, file/url)
GET        /v1/grades   userId, courseId
```

## Calendar Key APIs

```http
GET    /v1/calendars/:id/events?from=...&to=...
POST   /v1/events   calendar_id, title, start, end, recurrence?, attendees[]
PUT    /v1/events/:id
DELETE /v1/events/:id
GET    /v1/free-busy?user_ids=...&from=...&to=...   For scheduling
```

## LMS: Progress

- enrollment (user_id, course_id); progress (user_id, lesson_id, completed_at). getProgress(userId, courseId) = count(completed lessons) / total lessons.

## Calendar: Conflict

- Overlap: event1.start < event2.end AND event2.start < event1.end. getFreeSlots(attendeeIds, duration, range) = gaps in merged busy intervals (same as Meeting Scheduler).

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

