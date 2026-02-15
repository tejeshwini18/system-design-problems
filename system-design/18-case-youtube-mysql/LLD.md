# Low-Level Design: YouTube at Scale With MySQL

## 1. APIs (Representative)

```http
GET    /v1/videos/:id                 Video metadata + channel + counts (view, like)
GET    /v1/videos/:id/comments        Paginated comments
POST   /v1/videos                    Create video (upload flow; metadata first)
PUT    /v1/videos/:id/views          Increment view (or idempotent by session)
POST   /v1/videos/:id/like            Like / unlike
GET    /v1/channels/:id               Channel info + video list (paginated)
POST   /v1/subscriptions              Subscribe to channel
```

---

## 2. MySQL Schema (Per-Shard Conventions)

- **Shard key** in every table (e.g. `video_id` or `user_id`) so all queries can target one shard.

### Users / Channels (shard by user_id)
```sql
users (user_id PK, email, name, created_at);
channels (channel_id PK, user_id FK, name, subscriber_count BIGINT, updated_at);
```

### Videos (shard by video_id)
```sql
videos (
  video_id PK, channel_id FK, title, description, status ENUM(draft, processing, public),
  view_count BIGINT DEFAULT 0, like_count INT DEFAULT 0, comment_count INT DEFAULT 0,
  object_key VARCHAR, thumbnail_url VARCHAR, duration_sec INT, created_at
);
CREATE INDEX idx_channel_created ON videos(channel_id, created_at DESC);
```

### Engagement (same shard as video for comments/likes)
```sql
comments (comment_id PK, video_id FK, user_id FK, content TEXT, created_at);
CREATE INDEX idx_video_created ON comments(video_id, created_at DESC);

likes (user_id FK, video_id FK, PRIMARY KEY (user_id, video_id));
```

### Subscriptions (shard by subscriber user_id)
```sql
subscriptions (subscriber_id FK, channel_id FK, created_at, PRIMARY KEY (subscriber_id, channel_id));
```

---

## 3. Shard Resolution

```text
shard_id = hash(video_id) % num_shards   // for video, comments, likes
shard_id = hash(user_id) % num_shards    // for users, channels, subscriptions
```

- Application or middleware: extract shard key from request → choose DB connection (primary or replica) for that shard.
- Cross-shard: e.g. “subscribers of channel” — either fan-out at read (query subscriber shards by channel_id if indexed) or maintain a separate store (e.g. cache or table keyed by channel_id) updated by async job.

---

## 4. View Count (Avoid MySQL on Every View)

- **In memory / Redis:** key = `vc:video_id`, value = count; INCR on each view; TTL or none.
- **Periodic flush:** Every N seconds or M increments: read from Redis, UPDATE videos SET view_count = view_count + delta WHERE video_id = ?; then clear or decrement Redis.
- **Read:** view_count = Redis count + MySQL persisted count (or just Redis if you reconcile periodically from MySQL as baseline).

---

## 5. Key Classes / Modules

```text
VideoService
  - getVideo(videoId) → video, channel, counts  // shard by video_id; read from replica
  - createVideo(channelId, metadata) → videoId  // shard by new video_id; insert
  - incrementView(videoId, sessionId?)          // Redis INCR; async flush to MySQL

CommentService
  - listComments(videoId, limit, cursor)       // shard by video_id
  - addComment(videoId, userId, content)        // insert comment; UPDATE videos SET comment_count = comment_count + 1

SubscriptionService
  - subscribe(subscriberId, channelId)           // shard by subscriber_id
  - getSubscriptions(userId)                    // same shard
  - getSubscriberCount(channelId)               // cross-shard or cached counter
```

---

## 6. Error Handling

- **Shard down:** Return 503 for that shard; circuit breaker.
- **Replica lag:** Accept stale reads for counts; or read from primary for “after write” consistency.
- **Duplicate like:** Unique (user_id, video_id); return 200 with “already liked” or idempotent.

---

## 7. Index and Query Guidelines

- Always filter by shard key (e.g. video_id) so query hits one shard.
- Avoid cross-shard JOINs; do two queries and join in app if needed.
- Use covering indexes for list APIs (e.g. channel’s videos: index (channel_id, created_at DESC)).
- Keep hot counters out of critical path to MySQL; use cache + batch.
