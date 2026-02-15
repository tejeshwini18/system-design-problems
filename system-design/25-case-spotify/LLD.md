# Low-Level Design: How Spotify Works

## 1. APIs

```http
GET    /v1/tracks/:id                 Track metadata (title, artist, album, duration)
GET    /v1/tracks/:id/stream          Resolve stream URL (or manifest) + progress
PUT    /v1/me/player                  Set progress (position_ms, device_id)
GET    /v1/me/player                  Current playback state and queue
GET    /v1/search?q=...&type=track,artist,playlist   Search
GET    /v1/playlists/:id              Playlist metadata + track list
POST   /v1/playlists/:id/tracks       Add tracks (uris[], position?)
GET    /v1/recommendations             Home / Made for you (personalized)
GET    /v1/recommendations/next        Next track suggestion (context: current, queue)
```

---

## 2. Database Schemas (Conceptual)

```sql
tracks (track_id PK, title, duration_ms INT, artist_id FK, album_id FK, isrc, stream_path VARCHAR, regions JSON);
artists (artist_id PK, name, image_url);
albums (album_id PK, title, artist_id FK, image_url);
playlists (playlist_id PK, owner_id FK, name, collaborative BOOLEAN, updated_at);
playlist_tracks (playlist_id FK, track_id FK, position INT, added_at, added_by FK, PRIMARY KEY (playlist_id, position) or (playlist_id, track_id));
user_library (user_id FK, track_id FK, PRIMARY KEY (user_id, track_id));  -- liked
playback_progress (user_id FK, device_id, track_id FK, position_ms INT, updated_at, PRIMARY KEY (user_id, device_id));
```

---

## 3. Key Classes / Modules

```text
CatalogService
  - getTrack(trackId, region) → track metadata or 403 if not available
  - search(query, types, region, limit) → tracks, artists, playlists

PlaybackService
  - getStreamUrl(trackId, userId, region, quality) → url, expiry
  - saveProgress(userId, deviceId, trackId, positionMs)
  - getProgress(userId, deviceId) → current track, position

StreamUrlGenerator
  - signedUrl(trackId, quality, expiry) → CDN signed URL or manifest URL
  - path = tracks/{track_id}/{quality}/segment_0.ogg or manifest.m3u8

RecommendationService
  - getHomeFeed(userId, region) → playlists, albums, tracks (personalized)
  - getNextTrack(userId, currentTrackId, context) → trackId
  - use: listening history, likes, playlist membership; filter by region
```

---

## 4. Stream URL Generation

- **Option A (simple):** Pre-signed URL to object store or CDN: path = `tracks/{track_id}/{bitrate}.ogg` (or segment path); expiry 1 hour; client fetches directly.
- **Option B (adaptive):** Manifest URL (e.g. HLS): `stream/tracks/{track_id}/manifest.m3u8?token=...`; manifest lists segment URLs; client picks bitrate and fetches segments from CDN.
- **Region/license:** Before returning URL, check user region and track’s available_regions; return 403 if not allowed.

---

## 5. Progress Save

- Key: (user_id, device_id) or (user_id) if single device; value: track_id, position_ms, updated_at.
- PUT on pause, track change, or periodic (e.g. every 30 s) from client.
- Load on app open to resume.

---

## 6. Search (Elasticsearch)

- Index: tracks (title, artist_name, album_name, track_id); artists (name); playlists (name).
- Query: multi_match on title and artist; filter by region (available in user’s country); size and from for pagination.
- Autocomplete: suggest endpoint with prefix match or completion suggester.

---

## 7. Recommendations (Simplified Pipeline)

- **Offline:** Job: for each user, compute “Made for you” playlist (e.g. top 50 from model); store in cache or table (user_id → list of track_ids).
- **Online:** getHomeFeed() reads precomputed list; filter by region; return with metadata from catalog.
- **Next track:** Input = current track, recent history; model returns next track (or use rule: same playlist, related artist); filter by region; return one track_id.

---

## 8. Error Handling

- **Track not available in region:** 403 with message.
- **Stream URL expired:** Client requests new URL (same endpoint) before expiry.
- **Playlist conflict (collaborative):** Last-write-wins or operational transform for reorder; document in API.
