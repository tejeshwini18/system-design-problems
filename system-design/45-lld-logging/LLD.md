# LLD: Logging System (Chain of Responsibility)

## 1. Requirements

- **Log levels:** DEBUG, INFO, WARN, ERROR; configurable minimum level per app.
- **Handlers:** Console, File, Remote (send to server); each handler can have its own level and format.
- **Chain:** Log request passed along chain; each handler decides to handle (e.g. if level >= handler’s level) and/or pass to next.
- **Format:** Configurable (e.g. timestamp, level, message, context).
- **Optional:** Async write, rotation (file), batching (remote).

---

## 2. Core Classes (Chain of Responsibility)

```text
Logger – name, level; handlers: List<Handler>
  log(level, message, context?): if level >= this.level, for each handler: handler.handle(level, message, context)

Handler (abstract) – next: Handler?, minLevel
  handle(level, message, context): if level >= minLevel, doLog(level, message, context); if next != null, next.handle(...)
  doLog() – override in subclasses

ConsoleHandler extends Handler – doLog: print to stdout
FileHandler extends Handler – doLog: write to file (with rotation policy)
RemoteHandler extends Handler – doLog: send to log server (sync or queue)
```

---

## 3. Flow

1. Logger.log(INFO, "user logged in", { userId: 123 }).
2. Logger checks level >= logger.level; then calls handler1.handle(...).
3. Handler1 (e.g. ConsoleHandler): if INFO >= minLevel, doLog (print); then handler1.next.handle(...).
4. Handler2 (e.g. FileHandler): same; then next. Chain ends when next is null.

---

## 4. Design Patterns

- **Chain of Responsibility:** Handlers in chain; each can handle and/or pass to next.
- **Singleton:** Optional single root logger per process.
- **Strategy:** Formatter (JSON vs plain text) injectable per handler.

---

## 5. Key Methods

```text
Logger.log(level, message, context?)
Logger.addHandler(handler), setLevel(level)
Handler.setNext(handler), handle(level, message, context)
```

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

