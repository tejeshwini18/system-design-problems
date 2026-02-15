# HLD Concepts – Quick Reference

Concepts commonly asked in system design interviews (theory and when to use what).

---

## 1. Network Protocols

| Protocol | Layer | Use |
|----------|--------|-----|
| **TCP** | Transport | Reliable, ordered, connection-oriented; HTTP/HTTPS, DB connections. Handshake, flow control, retransmit. |
| **UDP** | Transport | Unreliable, no order guarantee; low latency (video, gaming, DNS). No handshake. |
| **HTTP/1.1** | Application | Request-response; stateless; keep-alive for reuse. |
| **HTTP/2** | Application | Multiplexing, binary, header compression; single connection many streams. |
| **WebSocket** | Application | Full-duplex over single TCP; handshake via HTTP then upgrade; chat, live updates. |
| **gRPC** | Application | HTTP/2 + Protobuf; streaming; microservices RPC. |

---

## 2. Client-Server vs P2P

- **Client-Server:** Central server; clients request; server authorizes and responds. Scale server; clients dumb. Example: Web, REST APIs.
- **P2P:** Peers communicate directly; no single server; discovery via DHT or tracker. Example: BitTorrent, blockchain nodes. Trade-off: no single point of failure vs complexity and churn.

---

## 3. CAP Theorem

- **C**onsistency: every read sees latest write. **A**vailability: every request gets response. **P**artition tolerance: system works despite network partition.
- **Theorem:** In presence of partition (P), choose **C** or **A** (not both in the strict sense).
- **CP:** Sacrifice availability under partition (e.g. wait for quorum). Example: etcd, ZooKeeper.
- **AP:** Sacrifice strong consistency under partition (e.g. allow stale read). Example: Dynamo-style, Cassandra.
- **CA:** Only in single-node or no partition; in practice **P** is assumed, so real choice is CP vs AP.

---

## 4. Microservices Patterns

- **SAGA:** Distributed transaction as sequence of local transactions; each has **compensating action** on failure. Choreography (events) or Orchestrator (central coordinator). Use when: cross-service transaction without 2PC.
- **Strangler Fig:** Gradually replace monolith by routing new features to new services; proxy routes by path/feature; over time more traffic to new system. Use when: incremental migration.

---

## 5. Scale 0 to Million Users (Phases)

- **0–10K:** Single server, single DB.
- **10K–100K:** Add LB, 2+ app servers, DB read replica, cache (Redis), static on CDN.
- **100K–1M:** Auto-scaling, multiple read replicas, async (queues), sharding or bigger DB.
- **1M+:** Sharding, event-driven, polyglot persistence, multi-region.

(Detailed in topic 20: AWS Scale to 10M.)

---

## 6. Consistent Hashing

- **Problem:** Distribute keys across N nodes; adding/removing node should move only ~K/N keys.
- **Ring:** Hash key and node ids to ring; key goes to first node clockwise. Virtual nodes (many points per physical node) for balance.
- **Use:** Cache sharding, load balancing, DHT.

(Detailed in topic 4: Distributed Cache.)

---

## 7. Back-of-Envelope Estimation

- **QPS:** DAU × requests per user per day / 86400; peak ≈ 2–3× average.
- **Storage:** rows × bytes per row × replication × retention.
- **Bandwidth:** QPS × request/response size.
- **Assumptions:** State them (e.g. 100M DAU, 10 requests/user/day).

---

## 8. SQL vs NoSQL – When to Use

| SQL | NoSQL |
|-----|--------|
| ACID, complex queries, JOINs, schema | Scale-out writes, flexible schema, key-value or document |
| User, order, transaction data | Sessions, cache, logs, time-series, high write |
| Vertical + read replicas first | Sharding + eventual consistency acceptable |
| Examples: Postgres, MySQL | Examples: Cassandra, MongoDB, Redis, DynamoDB |

---

## 9. Message Queue / Kafka (When to Use)

- **Decouple** producer and consumer; **buffer** burst; **async** processing; **replay** (Kafka log).
- **Patterns:** Task queue (SQS), event log (Kafka), pub/sub.
- (Detailed in topic 29: Apache Kafka.)

---

## 10. Proxy Servers

- **Forward proxy:** Client → proxy → origin (client identity hidden from origin; e.g. corporate proxy).
- **Reverse proxy:** Client → proxy → backend (backend hidden; LB, TLS termination, cache). Example: NGINX, ALB.
- **Use:** Load balancing, cache, auth, TLS offload, rate limiting.

---

## 11. CDN

- **Edge cache** for static and cacheable content; reduce latency and origin load.
- (Detailed in topic 36: CDN.)

---

## 12. Storage Types

| Type | Block | File | Object (S3) |
|------|--------|--------|-------------|
| Unit | Block (fixed size) | File/directory | Object (key + blob) |
| Use | VM disk, DB | NFS, shared FS | Blobs, backup, static assets |
| Access | OS/block protocol | File path | HTTP/key |
| **RAID** | Redundancy (striping, mirror, parity) for block storage; improve IO and fault tolerance. | | |

---

## 13. File System (GFS, HDFS)

- **GFS:** Single master (metadata); chunks (e.g. 64 MB) replicated (e.g. 3x); append-oriented; for batch analytics.
- **HDFS:** Similar; NameNode (metadata), DataNodes (chunks); write-once; MapReduce-style reads.
- **Use:** Large files, sequential read/write, throughput over latency.

---

## 14. Bloom Filter

- **Space-efficient** probabilistic structure: test "is element in set?" — may have **false positives**, no **false negatives**.
- **Use:** Avoid expensive lookups (e.g. "key not in DB" before disk read); spell check; crawl dedupe.
- **Operations:** add(key), mightContain(key); no delete in standard form (or use counting Bloom filter).

---

## 15. Merkle Tree & Gossip

- **Merkle tree:** Hash tree; root = hash of children; verify subset without full data; used in Git, blockchain, Dynamo.
- **Gossip protocol:** Nodes exchange state with random peers; eventual consistency of membership and data; used in Cassandra, member list propagation.

---

## 16. Caching

- **Cache aside:** App loads from DB on miss; writes to DB then invalidates/updates cache.
- **Write-through:** Write to cache and DB together.
- **Write-behind:** Write to cache; async flush to DB (risk of loss).
- **Eviction:** LRU, LFU, TTL; when full, evict by policy.
- (Detailed in topic 4: Distributed Cache; topic 36: CDN.)

---

## 17. Scaling Database

- **Read:** Read replicas; cache.
- **Write:** Vertical (bigger instance); **sharding** (partition by key).
- **Replication:** Async (eventual) vs sync (strong consistency, higher latency). **Leader election:** One primary; failover via election (etcd, Raft).
- **Partitioning:** Vertical (split columns) vs horizontal (shard rows). **Indexing:** B-tree, LSM; composite index for query pattern.
- (Sharding in topic 37: Database Sharding.)

---

## 18. Indexing

- **B-tree:** Range queries, point lookups; default in many RDBMS.
- **Hash index:** Point lookup only (no range).
- **LSM:** Write-optimized; compaction; used in Cassandra, RocksDB.
- **Secondary index:** Index on non-PK column; can be local (per shard) or global (separate store).
- **Covering index:** Index includes all columns needed by query; avoid table lookup.

---

## File Index (This Repo)

- **URL Shortening:** 02-url-shortener  
- **Rate Limiter:** 03-api-rate-limiter, 35-distributed-lock (for quotas)  
- **Search Autocomplete:** 10-search-autocomplete  
- **WhatsApp/Chat:** 01-scalable-chat-application  
- **Key-Value Store:** 04-distributed-cache, 30 Snowflake (IDs)  
- **Consistent Hashing:** 04-distributed-cache  
- **Scale 0 to Million:** 20-case-aws-scale-10m  
- **Notification:** 05-notification-system  
- **Pastebin:** 60-pastebin  
- **Twitter:** 27-case-twitter-timeline  
- **Dropbox:** 12-file-storage (sync aspects can extend)  
- **Instagram:** 32-instagram  
- **YouTube:** 07-video-streaming, 18-case-youtube-mysql  
- **Google Drive:** 12-file-storage  
- **Web Crawler:** 34-web-crawler  
- **News Feed:** 09-news-feed  
- **Ticketmaster:** 61-ticketmaster  
- **Nearby / Yelp:** 19-case-uber-nearby-drivers (geo), 06-ride-hailing  
- **Payment (LLD):** 08-payment-gateway  
- **Chat (LLD):** 01-scalable-chat-application  
- **Rate Limiter (LLD):** 03-api-rate-limiter  

All other LLDs (Parking Lot, Elevator, ATM, etc.) are in Part 4 of the main README.
