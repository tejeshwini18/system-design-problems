# Low-Level Design: Design Instagram

## 1. APIs

```http
POST   /v1/posts                    Create post (media_ids[], caption)
GET    /v1/feed                    Home feed (limit, cursor)
GET    /v1/users/:id/posts         User's posts (paginated)
GET    /v1/posts/:id                Single post
POST   /v1/posts/:id/like          Like / unlike
POST   /v1/posts/:id/comments       Add comment
GET    /v1/posts/:id/comments       List comments

POST   /v1/media/upload            Init upload; get upload_url
PUT    (upload_url)                 Upload bytes
POST   /v1/media/complete          Complete (media_ids[]) → use in post

POST   /v1/stories                  Create story (media_id)
GET    /v1/stories                  Stories from followees (who has story)
GET    /v1/stories/:user_id         User's story (current)
POST   /v1/stories/:id/view         Mark viewed (for view count)

GET    /v1/explore                  Explore feed (limit, cursor)
GET    /v1/search?q=...            Search users, hashtags
POST   /v1/follow/:id              Follow / unfollow
```

---

## 2. Database Schemas

```sql
users (user_id PK, username UNIQUE, display_name, avatar_url, created_at);
follows (follower_id FK, followee_id FK, created_at, PRIMARY KEY (follower_id, followee_id));

posts (post_id PK, user_id FK, caption TEXT, created_at);
post_media (post_id FK, media_id FK, position INT);
media (media_id PK, url VARCHAR, width, height, created_at);

likes (user_id FK, post_id FK, PRIMARY KEY (user_id, post_id));
comments (comment_id PK, post_id FK, user_id FK, text TEXT, created_at);
-- denormalize: posts.like_count, comment_count

stories (story_id PK, user_id FK, media_id FK, created_at);
story_views (story_id FK, user_id FK, viewed_at, PRIMARY KEY (story_id, user_id));
-- TTL or delete stories where created_at < now() - 24h
```

---

## 3. Key Classes / Modules

```text
PostService
  - createPost(userId, mediaIds, caption) → postId; fanOutToFollowers(postId, userId)
  - getFeed(userId, limit, cursor) → posts[], nextCursor
  - getPostsByUser(userId, limit, cursor) → posts[]

FeedCache
  - addToFeed(userId, postId, ts)   // for each follower
  - getFeed(userId, limit, cursor) → postIds[]
  - trim(userId, keepCount)

StoryService
  - createStory(userId, mediaId) → storyId; store with TTL 24h
  - getStoriesFromFollowees(userId) → list of (userId, storyId, mediaUrl, viewCount)
  - viewStory(storyId, userId)   // idempotent; increment view count

ExploreService
  - getExplore(userId, limit, cursor) → posts[]  // ranked; exclude seen
  - candidates: trending or personalized; rank; cache
```

---

## 4. Feed Read (Steps)

1. postIds = FeedCache.getFeed(userId, limit, cursor).
2. If cache miss or empty: pull from followees’ recent posts; merge sort; optionally backfill cache.
3. Hydrate: get posts (and media, like_count, comment_count, liked_by_me) by postIds from PostService.
4. Return ordered list and nextCursor (e.g. last post_id or ts).

---

## 5. Fan-out (Steps)

1. On new post: getFollowers(authorId).
2. If count > 500K: cap (e.g. active followers in last 30 days) or skip fan-out for this post (merge at read).
3. For each follower (batched): FeedCache.addToFeed(followerId, postId, ts).
4. Trim each feed to last 500 (ZREMRANGEBYRANK).

---

## 6. Story TTL

- **Option A:** Redis: key story:{user_id}, value story data, EXPIRE 86400 (24h). List “who has story” = SMEMBERS users_with_story (set refreshed by writer); get story per user from Redis.
- **Option B:** DB: cron every 1 min DELETE FROM stories WHERE created_at < NOW() - INTERVAL 24 HOUR. Read: SELECT * FROM stories WHERE user_id IN (followees) AND created_at > NOW() - 24h.

---

## 7. Explore Candidates

- **Trending:** Top posts by (likes + comments) in last 24h from global or by topic; store post_ids in cache; refresh every 5 min.
- **Personalized:** Embedding similarity (user’s liked posts vs candidate posts); or collaborative filtering. Batch job writes explore feed per user to cache; or real-time rank from candidate pool (e.g. 1K) and cache result.

---

## 8. Error Handling

- **Upload failure:** Retry; return 503. **Duplicate like:** Idempotent 200.
- **Story already expired:** 404. **Feed empty:** Return empty list and cursor null.
