# Low-Level Design: Video Streaming Platform

## 1. APIs

### Creator
```http
POST   /v1/videos/upload/init         Init multipart upload (filename, size) → upload_id, part_urls
PUT    /v1/videos/upload/part         Upload part (upload_id, part_number, body)
POST   /v1/videos/upload/complete     Complete upload (upload_id, parts[], title, description) → video_id
GET    /v1/videos/:id                 Video metadata and status (processing/ready)
DELETE /v1/videos/:id                 Delete video
```

### Viewer
```http
GET    /v1/videos/:id/play            Returns manifest URL (or redirect to CDN) + watch_token
GET    /v1/videos/:id                 Public metadata (title, creator, duration, thumbnail)
PUT    /v1/watch/progress             Update watch position (video_id, position_seconds)
GET    /v1/search?q=...              Search videos (paginated)
GET    /v1/recommendations            Personalized recommendations
```

---

## 2. Database Schemas

```sql
videos (
  video_id PK, creator_id FK, title, description, duration_seconds INT,
  status ENUM(uploading, processing, ready, failed), thumbnail_url,
  source_uri TEXT,  -- raw file in object store
  created_at, updated_at
);

video_renditions (
  video_id FK, quality VARCHAR(10),  -- 360p, 720p, 1080p
  manifest_url TEXT, segment_base_url TEXT,
  PRIMARY KEY (video_id, quality)
);

watch_progress (user_id FK, video_id FK, position_seconds INT, updated_at, PRIMARY KEY (user_id, video_id));

upload_jobs (job_id PK, video_id FK, source_uri, status ENUM(pending, running, done, failed), created_at);
```

---

## 3. Key Classes / Modules

```text
UploadService
  - initUpload(userId, filename, size) → uploadId, partUrls[]
  - completeUpload(uploadId, parts, metadata) → videoId  // writes to object store, creates video row, enqueues job

TranscodeWorker
  - pollJob() → job
  - process(job): download source → transcode (ffmpeg) → upload segments → write renditions → set status=ready

StreamingService
  - getPlayInfo(videoId, userId?) → manifestUrl, watchToken  // optional signed URL with expiry

ManifestGenerator (or static from transcode)
  - generateMasterManifest(videoId) → m3u8 with variant playlists per quality
  - variant manifest: list of segment URLs (e.g. seg_0.ts, seg_1.ts, ...)

ProgressService
  - updateProgress(userId, videoId, position)
  - getProgress(userId, videoId) → position
```

---

## 4. Transcode Pipeline (Steps)

1. Job = { video_id, source_uri }.
2. Download source from object store to worker disk.
3. Run ffmpeg (or similar):
   - Output 360p, 720p, 1080p to temp files; segment with 4 s segment duration (HLS: `-hls_time 4 -hls_list_size 0`).
4. Upload segment files to object store under path: `videos/{video_id}/{quality}/seg_0.ts`, ...
5. Generate master.m3u8 and quality-specific .m3u8; upload to object store.
6. Insert/update video_renditions with manifest_url (CDN URL); set videos.status = ready.
7. Delete temp files and optional raw source (or move to cold storage).

---

## 5. Manifest (HLS) Structure

```text
master.m3u8:
  #EXTM3U
  #EXT-X-STREAM-INF:BANDWIDTH=500000,RESOLUTION=640x360
  https://cdn.../videos/v123/360p/playlist.m3u8
  #EXT-X-STREAM-INF:BANDWIDTH=2000000,RESOLUTION=1280x720
  https://cdn.../videos/v123/720p/playlist.m3u8

360p/playlist.m3u8:
  #EXTM3U
  #EXTINF:4.0,
  seg_0.ts
  #EXTINF:4.0,
  seg_1.ts
  ...
```

---

## 6. Play URL and Security

- Option A: Public CDN URLs for manifest and segments (unlisted = no listing, share link only).
- Option B: Signed URL (HMAC or JWT) with expiry (e.g. 1 h) for manifest; segments under same path signed or cookie.
- getPlayInfo returns signed manifest URL or redirect to CDN with token in query.

---

## 7. Error Handling

- Upload part failure: retry; complete only when all parts received.
- Transcode failure: set status=failed; notify creator; optional retry from queue.
- Missing rendition: return 404 or fallback to lower quality.

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

