# Low-Level Design: How Reddit Works

## 1. APIs

```http
GET    /v1/subreddits/:name/posts?sort=hot&limit=25&after=t3_xxx
GET    /v1/posts/:id                 Post detail + top-level comments (sort=best)
GET    /v1/posts/:id/comments        Comments (flat or tree; parent_id for "load more")
POST   /v1/posts                     Create post (subreddit_id, title, body)
POST   /v1/comments                  Create comment (post_id, parent_id?, body)
POST   /v1/vote                      Vote (item_id, direction: 1 | -1 | 0 to remove)
GET    /v1/feed/home?sort=hot        Home feed (subscribed subreddits)
POST   /v1/subreddits/:id/subscribe
DELETE /v1/subreddits/:id/subscribe
```

---

## 2. Database Schemas

```sql
subreddits (subreddit_id PK, name UNIQUE, description, created_at);
subscriptions (user_id FK, subreddit_id FK, PRIMARY KEY (user_id, subreddit_id));

posts (post_id PK, subreddit_id FK, author_id FK, title, body TEXT, score INT DEFAULT 0, hot_score FLOAT, created_at);
CREATE INDEX idx_sub_created ON posts(subreddit_id, created_at DESC);
CREATE INDEX idx_sub_score ON posts(subreddit_id, score DESC);
CREATE INDEX idx_sub_hot ON posts(subreddit_id, hot_score DESC);

comments (comment_id PK, post_id FK, parent_id FK NULL, author_id FK, body TEXT, score INT DEFAULT 0, created_at);
CREATE INDEX idx_post_parent ON comments(post_id, parent_id, score DESC);
CREATE INDEX idx_post_created ON comments(post_id, parent_id, created_at DESC);

votes (user_id FK, item_id FK, direction SMALLINT,  -- +1 or -1  PRIMARY KEY (user_id, item_id));
-- item_id can be post_id or comment_id; use polymorphic or separate tables
```

---

## 3. Key Classes / Modules

```text
PostService
  - createPost(subredditId, authorId, title, body) → postId
  - getPostsBySubreddit(subredditId, sort, limit, after) → posts[]
  - getPost(postId) → post

CommentService
  - addComment(postId, parentId, authorId, body) → commentId
  - getCommentsByPost(postId, sort, limit, parentId?) → comments[]  // parentId null = top-level

VoteService
  - vote(userId, itemId, direction)  // upsert; update score
  - getScore(itemId) → score

ScoreUpdater
  - onVote(itemId, delta): Redis INCRBY score:itemId delta; or UPDATE posts SET score = score + delta
  - hot score: periodic job or on vote: hot = f(score, age)

FeedService
  - getHomeFeed(userId, sort, limit, after) → posts[]
  - getSubscribedSubreddits(userId) → subredditIds[]
  - mergePostsBySubreddit(subredditIds, sort, limit) → postIds[]; hydrate
```

---

## 4. Hot Score (Formula)

- **Reddit-style:** hot = log10(max(|score|, 1)) + (created_at - epoch) / 45000. Higher = hotter. Store and index hot_score; update periodically (e.g. every minute) for recent posts, or on each vote (expensive).
- **Simplified:** hot_score = score / (1 + age_hours) or score * exp(-decay * age). Update in background job.

---

## 5. Comment Tree Load

1. Top-level: SELECT * FROM comments WHERE post_id = ? AND parent_id IS NULL ORDER BY score DESC LIMIT 20.
2. Client shows 20; "load more" for one thread: SELECT * FROM comments WHERE parent_id = ? ORDER BY score DESC LIMIT 20.
3. Or: single query all comments for post_id; build tree in app; return tree or flat with depth; paginate by returning top-level first and lazy-load children.

---

## 6. Vote (Steps)

1. Upsert votes (user_id, item_id, direction). If previous direction exists, delta = new - old; else delta = new.
2. Update score: Redis INCRBY score:item_id delta; or UPDATE posts SET score = score + ? WHERE post_id = ?.
3. If hot_score depends on score: enqueue job to recompute hot_score for item (or do synchronously for small scale).
4. Return new score to client.

---

## 7. Feed Merge (Steps)

1. subreddit_ids = getSubscribedSubreddits(userId).
2. For each subreddit_id, get post list (from cache or DB): getPostsBySubreddit(subreddit_id, sort, limit=50).
3. Merge: combine lists; sort by hot_score or created_at; take top 25; cursor = last post_id or (hot_score, post_id).
4. Hydrate post details; return.

---

## 8. Error Handling

- **Duplicate vote same direction:** Idempotent; return 200 with current score.
- **Post not in subreddit:** 403.
- **Comment parent not found:** 400.
- **Rate limit:** 429 on vote and post creation.
