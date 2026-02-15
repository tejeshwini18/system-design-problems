# Low-Level Design: Twitter Timeline

## 1. APIs

```http
POST   /v1/tweets                    Create tweet (text, reply_to_id?)
GET    /v1/timeline/home             Home timeline (limit=20, cursor?)
GET    /v1/tweets/:id                Single tweet (with replies thread optional)
POST   /v1/tweets/:id/like           Like / unlike
POST   /v1/tweets/:id/retweet       Retweet
GET    /v1/users/:id/tweets          User's tweets (paginated)
POST   /v1/follow/:id               Follow user
DELETE /v1/follow/:id                Unfollow
```

---

## 2. Database Schemas

```sql
tweets (tweet_id PK, user_id FK, text VARCHAR(280), created_at, reply_to_id FK NULL, retweet_of_id FK NULL);
CREATE INDEX idx_user_created ON tweets(user_id, created_at DESC);

follows (follower_id FK, followee_id FK, created_at, PRIMARY KEY (follower_id, followee_id));
CREATE INDEX idx_followee ON follows(followee_id);  -- for fan-out: get followers

likes (user_id FK, tweet_id FK, PRIMARY KEY (user_id, tweet_id));
-- counts: denormalized in tweets (like_count, retweet_count) or separate counts table
```

---

## 3. Redis Timeline Cache (Push)

```text
Key: timeline:{user_id}   Type: Sorted Set   Score: timestamp (or ranking score)   Member: tweet_id
  ZADD timeline:u123 1707890123 tweet_456
  ZREVRANGE timeline:u123 0 19   -- top 20
  ZREMRANGEBYRANK timeline:u123 0 -801   -- keep last 800 (trim from bottom)
```

---

## 4. Key Classes / Modules

```text
TweetService
  - createTweet(userId, text, replyToId?) → tweetId
  - getTweet(tweetId) → tweet
  - getTweetsByAuthor(userId, limit, cursor) → tweets[]

TimelineService
  - getHomeTimeline(userId, limit, cursor) → tweets[], nextCursor
  - read from Redis timeline:userId; hydrate tweet_ids from TweetStore

FanOutWorker
  - onTweetCreated(authorId, tweetId, ts): getFollowers(authorId); for each f: ZADD timeline:f ts tweetId; trim
  - getFollowers(userId): from SocialGraph (DB or cache)

SocialGraphService
  - getFollowees(userId) → userIds[]
  - getFollowers(userId) → userIds[]   // for fan-out
  - follow(followerId, followeeId); unfollow(...)
```

---

## 5. Timeline Read (Steps)

1. cursor = decode(cursor) → last_timestamp or null.
2. If cursor: ZREVRANGEBYSCORE timeline:user_id -inf (cursor_exclusive) LIMIT 0 limit+1; else ZREVRANGE timeline:user_id 0 limit.
3. List of tweet_ids (and scores); batch get tweets from TweetStore (or cache).
4. Filter (e.g. author blocked); build nextCursor from last item if we have limit+1.
5. Return { tweets, nextCursor }.

---

## 6. Fan-out (Steps)

1. Consume event { author_id, tweet_id, ts }.
2. followers = getFollowers(author_id). If len(followers) > 100000: optionally skip or fan-out to subset (e.g. active in last 7 days).
3. Pipeline: for each follower_id in followers: ZADD timeline:follower_id ts tweet_id.
4. Trim: for each affected user, ZREMRANGEBYRANK timeline:user_id 0 -801 (keep 800 most recent). Can do async or in batches.

---

## 7. Cursor

- nextCursor = base64(last_tweet_id + ":" + last_ts) so next request uses ZREVRANGEBYSCORE with maxScore < last_ts.
- Client sends cursor in next request; server decodes and queries.

---

## 8. Error Handling

- **Tweet too long:** 400.
- **Duplicate like:** Idempotent; return 200.
- **Timeline empty:** Return empty list; nextCursor null.
- **Redis down:** Fallback to pull (get followees, get their recent tweets, merge); slower.
