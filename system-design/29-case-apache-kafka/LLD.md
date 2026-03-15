# Low-Level Design: How Apache Kafka Works

## 1. Key Concepts (LLD Level)

### Topic and Partition
- **Topic:** name (e.g. "orders"); config: num_partitions, replication_factor, retention.ms, retention.bytes.
- **Partition:** id within topic (0 to P-1); each partition is an ordered log (offset 0, 1, 2, ...); stored on one leader broker and replicated to N-1 followers.

### Message
- **Record:** key (optional), value (bytes), timestamp, headers, partition (assigned by broker or producer).
- **Offset:** monotonic per partition; consumer reads from offset; broker returns batch (offset, record) per partition.

### Producer
- **Config:** bootstrap.servers, acks (0, 1, all), retries, idempotence (exactly-once), key.serializer, value.serializer.
- **Send:** producer.send(ProducerRecord(topic, key, value)) → Future; partition = hash(key) % num_partitions (default).
- **Batch:** Records buffered by partition; send when batch.size or linger.ms; compression optional.

### Consumer
- **Config:** bootstrap.servers, group.id, auto.offset.reset (earliest/latest), enable.auto.commit.
- **Subscribe:** consumer.subscribe(["topic"]); poll() returns ConsumerRecords (per partition, list of records).
- **Commit:** consumer.commitSync() or commitAsync() to persist offset for group+partition; next poll continues from committed offset.

---

## 2. Partition Assignment (Consumer Group)

- **Protocol:** Range, RoundRobin, or Sticky (and CooperativeSticky). Coordinator assigns partitions to members; each member gets list of (topic, partition).
- **Rebalance:** Triggered by join (new consumer), leave (heartbeat timeout), or explicit. All members stop consuming; get new assignment; resume from committed offset.
- **Coordinator:** Broker that hosts __consumer_offsets partition for group_id hash; consumer finds coordinator via FindCoordinator request.

---

## 3. Replication Protocol (Simplified)

- **Leader:** Accepts produce and fetch for partition. Replicates to followers (same order).
- **Follower:** Replicates from leader; sends fetch request; appends to local log; reports high water mark (HW).
- **ISR:** Leader tracks replicas that are “in sync” (caught up within lag threshold). Leader only acks produce when all ISR have written (for acks=all).
- **Leader election:** Controller selects new leader from ISR when leader fails; metadata updated; producers/consumers retry with new leader.

---

## 4. Log Segment (On Disk)

- **Directory:** topic-partition (e.g. orders-0)/ segment files: 00000000000000000000.log, .index, .timeindex.
- **.log:** Sequential append of records (offset, size, key, value, ...). No in-place update; delete by removing old segments (retention) or compaction.
- **.index:** Sparse index: offset → byte position in .log for fast seek(offset).
- **Fetch:** Given offset, find segment (binary search by offset range); look up position in .index; read from .log.

---

## 5. Key Classes / Modules (Conceptual)

```text
Broker
  - PartitionManager: getLeader(partition), getReplica(partition)
  - LogManager: append(partition, records), fetch(partition, offset, maxBytes)
  - ReplicaManager: replicate to followers; update ISR

Producer (client)
  - partitioner: partition(topic, key, value, cluster) → partition
  - accumulator: buffer by (topic, partition); batch
  - sender: send batches to leader; handle acks and retries

Consumer (client)
  - coordinator: join group; receive assignment; commit offset
  - fetcher: fetch from assigned partitions (from committed offset)
  - subscription: assigned partitions; poll() returns records
```

---

## 6. Produce Request (Wire)

- **Request:** topic, partition, records (batch); acks; timeout.
- **Response:** partition, base_offset (first offset of batch), error_code. Producer retries on error (e.g. NOT_LEADER → refresh metadata and retry).

---

## 7. Fetch Request (Wire)

- **Request:** topic, partition, offset, max_bytes; consumer group (for offset commit and lag).
- **Response:** records (offset, key, value, ...) per partition; high_water_mark. Consumer commits offset after process; next fetch uses committed offset.

---

## 8. Error Handling

- **NOT_LEADER_FOR_PARTITION:** Refresh metadata; retry to new leader.
- **OFFSET_OUT_OF_RANGE:** Consumer auto.offset.reset: earliest or latest; or fail.
- **REBALANCE_IN_PROGRESS:** Consumer stops current fetch; waits for new assignment; resumes.
- **Duplicate produce (idempotent):** Producer uses PID and sequence; broker deduplicates; at-most-once per key in idempotent mode when combined with transactions.
