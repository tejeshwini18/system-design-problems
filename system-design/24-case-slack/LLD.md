# Low-Level Design: How Slack Works

## 1. APIs

```http
POST   /v1/chat.postMessage           Send message (channel, text, thread_ts?)
GET    /v1/conversations.history     List messages (channel_id, limit, cursor)
GET    /v1/conversations.replies     List thread replies (channel_id, thread_ts, limit)
POST   /v1/reactions.add             Add emoji reaction
GET    /v1/users.getPresence          Get user presence
GET    /v1/search.messages            Full-text search (query, channel?, user?, count)
```

**WebSocket:** `WS /v1/rtm?token=...`  
- Client → Server: `{ "type": "message", "channel": "C123", "text": "..." }`; `{ "type": "typing", "channel": "C123" }`  
- Server → Client: `{ "type": "message", ... }`; `{ "type": "presence_change", "user": "...", "presence": "active" }`; `{ "type": "user_typing", ... }`

---

## 2. Database Schemas

```sql
channels (channel_id PK, workspace_id FK, name, type ENUM(public, private, im), created_at);
channel_members (channel_id FK, user_id FK, joined_at, PRIMARY KEY (channel_id, user_id));

messages (
  message_id PK, channel_id FK, user_id FK, text TEXT, ts DECIMAL UNIQUE,
  thread_ts DECIMAL NULL,  -- null for root; set for replies
  edited_at, deleted_at, created_at
);
CREATE INDEX idx_channel_ts ON messages(channel_id, ts DESC);
CREATE INDEX idx_channel_thread ON messages(channel_id, thread_ts, ts) WHERE thread_ts IS NOT NULL;
```

---

## 3. Key Classes / Modules

```text
MessageService
  - sendMessage(channelId, userId, text, threadTs?) → message
  - getHistory(channelId, limit, cursor) → messages[]
  - getThreadReplies(channelId, threadTs, limit) → messages[]

PresenceService
  - setPresence(userId, presence)
  - getPresence(userIds[]) → map[userId]presence
  - setTyping(userId, channelId, isTyping)

RealtimeHub
  - register(connectionId, userId, channelIds[])   // which channels this connection cares about
  - broadcastToChannel(channelId, payload)
  - sendToUser(userId, payload)

SearchService
  - indexMessage(message)
  - search(workspaceId, query, filters, limit) → hits[]
```

---

## 4. Message Send (Steps)

1. Resolve channel_id and check user is member.
2. Generate ts (e.g. epoch float or snowflake); insert message (channel_id, user_id, text, ts, thread_ts).
3. Publish to Kafka/SQS: topic/queue keyed by channel_id, payload = message.
4. RealtimeHub: for each server that has subscribers in this channel, forward payload (or each server consumes from queue for channels it serves); push to WebSocket connections.
5. Notification worker: consume; if message has @user or user in channel with notify on all, send push to those users (except sender and online).
6. Search indexer: consume; index message in Elasticsearch (workspace_id, channel_id, user_id, text, ts).

---

## 5. Cursor Pagination (History)

- cursor = base64(ts of last message from previous page).
- Query: SELECT * FROM messages WHERE channel_id = ? AND ts < cursor ORDER BY ts DESC LIMIT 21.
- Return 20 items and next_cursor = ts of 20th (or null if < 21 rows).

---

## 6. Presence (Redis)

```text
Key: presence:{user_id}   Value: active|away|dnd   TTL: 60s
Key: typing:{channel_id}  Hash: user_id → 1   TTL per field: 5s
```
- Heartbeat every 30s: SET presence:{user_id} active EX 60.
- Typing: HSET typing:channel_id user_id 1; EXPIRE field or key 5s; broadcast to channel; client clears on blur or after 5s.

---

## 7. Error Handling

- **Not in channel:** 403.
- **Channel not found:** 404.
- **Rate limit:** 429 per user and per channel.
- **WebSocket disconnect:** Unregister from RealtimeHub; presence set offline or TTL expire.
