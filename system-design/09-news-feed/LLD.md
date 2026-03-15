# Low-Level Design: News Feed System

## 1. APIs

```http
POST   /v1/posts                    Create post (content, media_urls[])
GET    /v1/posts/:id                 Get single post
GET    /v1/feed                     Get feed (limit=20, cursor=optional)
POST   /v1/follow/:userId           Follow user
DELETE /v1/follow/:userId            Unfollow user
POST   /v1/posts/:id/like            Like post
POST   /v1/posts/:id/comments        Add comment
GET    /v1/posts/:id/comments        List comments (paginated)
```

---

## 2. Database Schemas

```sql
posts (post_id PK, user_id FK, content TEXT, created_at, like_count INT DEFAULT 0, comment_count INT DEFAULT 0);
CREATE INDEX idx_user_created ON posts(user_id, created_at DESC);

follows (follower_id FK, followee_id FK, created_at, PRIMARY KEY (follower_id, followee_id));
CREATE INDEX idx_followee ON follows(followee_id);  -- for fan-out: get followers of followee

feed_entries (user_id FK, post_id FK, created_at, PRIMARY KEY (user_id, post_id));
-- Or use Redis: ZADD feed:user_id timestamp post_id
CREATE INDEX idx_user_created ON feed_entries(user_id, created_at DESC);

likes (user_id FK, post_id FK, PRIMARY KEY (user_id, post_id));
comments (comment_id PK, post_id FK, user_id FK, content TEXT, created_at);
```

---

## 3. Key Classes / Modules

```text
PostService
  - createPost(userId, content, mediaUrls) → postId
  - getPost(postId) → post
  - getPostsByIds(postIds) → posts[]  // batch for feed

FeedService
  - getFeed(userId, limit, cursor) → { items: post[], nextCursor }
  - warmCache(userId)  // on login or first request

FanOutWorker
  - onPostCreated(postId, authorId, timestamp): getFollowers(authorId) → for each f: addToFeed(f, postId, timestamp)

SocialGraphService
  - follow(followerId, followeeId)
  - unfollow(followerId, followeeId)
  - getFollowers(userId) → userIds[]   // for fan-out
  - getFollowees(userId) → userIds[]   // for pull merge

FeedRepository (Redis)
  - addToFeed(userId, postId, timestamp)   // ZADD feed:userId timestamp postId
  - getFeed(userId, limit, maxScore?) → [(postId, score)]   // ZREVRANGEBYSCORE with limit
  - trimFeed(userId, keepCount)   // ZREMRANGEBYRANK to keep latest N
```

---

## 4. Feed Read (Steps)

1. cursor = decode(cursor) → (last_post_id, last_ts) or null for first page.
2. Redis: ZREVRANGEBYSCORE feed:userId -inf (last_ts or +inf) LIMIT 0 (limit+1). If no cursor, ZREVRANGE feed:userId 0 (limit).
3. If result empty or small and no cursor: optional pull merge (get followees, get their recent posts, merge sort, push top N to Redis for next time).
4. Resolve post_ids to posts: PostService.getPostsByIds(postIds).
5. Build nextCursor from last item if we have limit+1 items.
6. Return { items: posts, nextCursor }.

---

## 5. Fan-out (Steps)

1. Receive event { post_id, author_id, created_at }.
2. SocialGraphService.getFollowers(author_id) → list (paginated if huge).
3. If count > 10000: optionally skip push for this post (celebrity) or fan-out in batches with delay.
4. For each follower (in batches of 500): FeedRepository.addToFeed(follower_id, post_id, created_at); trimFeed(follower_id, 1000).
5. Use pipeline or Lua to batch Redis writes.

---

## 6. Cursor Encoding

- nextCursor = base64(last_post_id + ":" + last_created_at_ts) so client sends back for next page.
- Decode to get (last_post_id, last_ts); query feed with maxScore < last_ts (exclusive) for next page.

---

## 7. Cache Miss (Cold User or New Account)

- getFollowees(userId) → get recent posts per followee (e.g. 50 each); merge by created_at; take top 20; optionally backfill Redis for next 5 pages.

---

## 8. Error Handling

- Post not found: 404.
- Feed empty: return empty list and no nextCursor.
- Redis down: fallback to pull merge from DB (slower); or 503.

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

