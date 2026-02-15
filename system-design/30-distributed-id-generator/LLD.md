# Low-Level Design: Distributed ID Generator (Snowflake)

## 1. API

```text
IdGenerator.nextId() → long   // blocking; thread-safe
IdGenerator.nextIds(count: int) → long[]   // batch for throughput
```

- No REST API required for core logic; service exposes HTTP if needed: `GET /v1/id` or `GET /v1/ids?count=100`.

---

## 2. Bit Layout Constants

```text
NODE_ID_BITS = 10
SEQUENCE_BITS = 12
TIMESTAMP_BITS = 41
EPOCH_MS = 1609459200000   // 2021-01-01 00:00:00 UTC

MAX_SEQUENCE = (1 << SEQUENCE_BITS) - 1   // 4095
MAX_NODE_ID = (1 << NODE_ID_BITS) - 1     // 1023
```

---

## 3. Key Classes / Modules

```text
SnowflakeIdGenerator
  - nodeId: int
  - lastTimestamp: long
  - sequence: int
  - lock: Mutex (or atomic CAS for lock-free)
  - nextId() → long
  - nextIds(n) → long[]

BitPacker
  - pack(timestampMs, nodeId, sequence) → long
    id = ((timestampMs - EPOCH_MS) << (NODE_ID_BITS + SEQUENCE_BITS))
       | (nodeId << SEQUENCE_BITS)
       | sequence
  - parse(id) → (timestampMs, nodeId, sequence)   // for debugging
```

---

## 4. nextId() Algorithm

```text
loop:
  nowMs = currentTimeMillis()
  if nowMs < lastTimestamp:
    handleClockBackward(nowMs)   // wait or throw
  if nowMs > lastTimestamp:
    sequence = 0
    lastTimestamp = nowMs
  if sequence > MAX_SEQUENCE:
    waitUntilNextMs()
    continue
  id = pack(nowMs - EPOCH_MS, nodeId, sequence)
  sequence++
  return id
```

---

## 5. Clock Backward Handling

```text
handleClockBackward(nowMs):
  diff = lastTimestamp - nowMs
  if diff <= MAX_CLOCK_SKEW_MS:   // e.g. 5ms
    Thread.sleep(diff)   // wait until time catches up
  else:
    throw new ClockMovedBackwardException()
  // or: use lastTimestamp + 1 and log warning (weak ordering)
```

---

## 6. Thread Safety

- **Option A:** synchronized block or ReentrantLock around (lastTimestamp, sequence) update and pack.
- **Option B:** AtomicLong for sequence; CAS loop; lastTimestamp in volatile; ensure only one thread advances time/sequence per ms.
- **Option C:** Single-threaded ID generator thread; others take from queue (higher latency, simpler).

---

## 7. Node ID at Startup

```text
nodeId = getFromEnv("NODE_ID")   // or
nodeId = coordinator.getOrCreateNodeId()   // ZooKeeper: /id-gen/nodes create sequential
nodeId = hash(instanceId) % (MAX_NODE_ID + 1)
if nodeId not unique: retry with different seed or fail fast
```

---

## 8. Error Handling

- **Clock moved backward:** Retry after sleep or return error; alert.
- **Sequence overflow in same ms:** Wait for next ms (rare if 4096/ms enough).
- **Node ID collision:** Startup check or coordinator; fail fast before generating IDs.
