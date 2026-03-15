# Low-Level Design: Leaderboard System

## 1. APIs

```http
POST   /v1/leaderboards/:id/scores
  Body: { "user_id": "u123", "score": 1500 }   // set absolute
  Or:    { "user_id": "u123", "delta": 100 }   // increment
  Response: { "user_id": "u123", "score": 1500, "rank": 42 }

GET    /v1/leaderboards/:id/top?limit=100&offset=0
  Response: {
    "entries": [
      { "rank": 1, "user_id": "u1", "score": 2000, "display_name": "..." },
      ...
    ]
  }

GET    /v1/leaderboards/:id/rank/:userId
  Response: { "user_id": "u123", "score": 1500, "rank": 42 }

GET    /v1/leaderboards/:id/scores/:userId
  Response: { "user_id": "u123", "score": 1500, "rank": 42 }
```

---

## 2. Redis Key Design

```text
Key:   lb:{leaderboard_id}
       e.g. lb:global, lb:game:1, lb:game:1:daily:20240214
Type:  Sorted Set
Member: user_id
Score:  score (integer; higher = better)
Commands:
  ZADD lb:global 1500 u123
  ZREVRANGE lb:global 0 99 WITHSCORES    -- top 100
  ZREVRANK lb:global u123                -- 0-based rank
  ZSCORE lb:global u123                  -- score
  ZINCRBY lb:global 100 u123             -- add 100 to score
```

---

## 3. Key Classes / Modules

```text
LeaderboardService
  - updateScore(leaderboardId, userId, score)  // set
  - incrementScore(leaderboardId, userId, delta)
  - getTop(leaderboardId, limit, offset) → entries[]
  - getRank(leaderboardId, userId) → rank, score
  - getScore(leaderboardId, userId) → score

LeaderboardRepository (Redis)
  - zadd(key, score, member)
  - zrevrange(key, start, stop, withScores) → [(member, score)]
  - zrevrank(key, member) → rank 0-based
  - zscore(key, member) → score
  - zincrby(key, delta, member)

UserEnrichment
  - getDisplayNames(userIds[]) → map[userId]displayName  // from DB or cache
```

---

## 4. Get Top (Steps)

1. key = "lb:" + leaderboardId.
2. ZREVRANGE key offset (offset+limit-1) WITHSCORES → [(user_id, score), ...].
3. rank = offset + 1 for first element.
4. Batch get display names for user_ids (UserEnrichment.getDisplayNames).
5. Build response: { rank, user_id, score, display_name } for each; return.

---

## 5. Update Score and Return Rank

1. ZADD key score user_id (or ZINCRBY key delta user_id).
2. newScore = ZSCORE key user_id.
3. rank = ZREVRANK key user_id; displayRank = rank + 1.
4. Return { user_id, score: newScore, rank: displayRank }.

---

## 6. Multiple Periods (Daily/Weekly)

- On score update: update both "lb:game:1" (all-time) and "lb:game:1:daily:YYYYMMDD" (daily).
- Client or cron at midnight: create new key for new day; old daily key can expire (EXPIRE 7 days) or be archived.
- getTop(leaderboardId, period="daily") → use key with date suffix.

---

## 7. Friend Leaderboard (Optional)

- Get friend IDs for user (from social graph).
- Pipeline: for each friend_id, ZSCORE lb:global friend_id.
- Build list (friend_id, score); sort by score descending; assign rank 1, 2, ...; optionally merge with global rank (ZREVRANK for each).
- Return list with rank and score.

---

## 8. Snapshot for History

- Cron: daily at 00:05, ZREVRANGE lb:game:1 0 9999 WITHSCORES; insert into leaderboard_snapshots (date, leaderboard_id, rank, user_id, score).
- Query historical leaderboard: SELECT * FROM leaderboard_snapshots WHERE leaderboard_id = ? AND date = ? ORDER BY rank.

---

## 9. Error Handling

- Leaderboard not found: 404 (invalid leaderboard_id).
- User not in set: getRank returns null or rank 0; getScore returns 0 or 404.
- Negative score: allow or clamp to 0 per product.
