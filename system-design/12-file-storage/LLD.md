# Low-Level Design: File Storage System

## 1. APIs

```http
POST   /v1/files/upload/init
  Body: { "name": "doc.pdf", "parent_id": "folder_123", "size": 10485760 }
  Response: { "upload_id": "u_xxx", "file_id": "f_yyy", "part_size": 5242880, "part_urls": [{"part": 1, "url": "..."}, ...] }

PUT     (presigned URL from part_urls)     Upload one part (body = binary)
POST   /v1/files/upload/complete
  Body: { "upload_id": "u_xxx", "parts": [{"part": 1, "etag": "..."}, ...] }
  Response: { "file_id": "f_yyy", "size", "status": "ready" }

GET    /v1/files/:id                     Metadata (name, size, mime_type, created_at, ...)
GET    /v1/files/:id/download            Redirect to presigned URL or return URL
GET    /v1/files?parent_id=...           List children (paginated)
POST   /v1/folders                       Create folder (name, parent_id)
PATCH  /v1/files/:id                     Rename, move (parent_id)
DELETE /v1/files/:id                     Soft delete or move to trash
POST   /v1/files/:id/share               Share (user_id or link, permission)
GET    /v1/files/:id/versions            List versions
POST   /v1/files/:id/restore            Restore version (version_id)
```

---

## 2. Database Schemas

```sql
files (
  file_id PK, parent_id FK NULL, owner_id FK, name VARCHAR, type ENUM(file, folder),
  size BIGINT, object_key VARCHAR NULL, mime_type VARCHAR, version INT DEFAULT 1,
  status ENUM(uploading, ready, deleted), created_at, updated_at
);
CREATE INDEX idx_parent ON files(parent_id);
CREATE INDEX idx_owner ON files(owner_id);
CREATE UNIQUE INDEX idx_parent_name ON files(parent_id, name) WHERE status != 'deleted';

file_versions (file_id FK, version_id PK, object_key, size BIGINT, created_at);

shares (id PK, resource_id FK, resource_type ENUM(file, folder), shared_with VARCHAR, permission ENUM(read, write), created_at);
CREATE INDEX idx_resource ON shares(resource_id);
CREATE INDEX idx_shared_with ON shares(shared_with);

upload_sessions (upload_id PK, file_id FK, part_size INT, status ENUM(active, completed), created_at);
```

---

## 3. Key Classes / Modules

```text
MetadataService
  - createFile(ownerId, parentId, name, type, size?) → fileId
  - getFile(fileId, userId) → file  // with permission check
  - listChildren(parentId, userId, limit, cursor) → items[]
  - move(fileId, newParentId, userId)
  - rename(fileId, name, userId)
  - delete(fileId, userId)

UploadService
  - initUpload(userId, parentId, name, size) → uploadId, fileId, partUrls[]
  - completeUpload(uploadId, parts[])  // merge in object store, update file object_key, size, status

DownloadService
  - getDownloadUrl(fileId, userId) → presignedUrl  // permission check, then sign object_key

PermissionService
  - canRead(userId, fileId) → boolean
  - canWrite(userId, fileId) → boolean
  // Resolve: owner or shares where resource in (file, ancestors) and permission >= read

ObjectStore (interface)
  - createMultipartUpload(key) → uploadId
  - getPresignedUploadUrl(uploadId, key, partNumber) → url
  - completeMultipartUpload(uploadId, key, parts[]) 
  - getPresignedDownloadUrl(key, expiry) → url
```

---

## 4. Permission Resolution

- If user_id == file.owner_id → allow.
- Else: find shares where resource_id = file_id or resource_id in ancestor folder ids and (shared_with = user_id or shared_with = link_id). If any with read → canRead; with write → canWrite.
- Cache: (user_id, file_id) → permission with short TTL to reduce DB load.

---

## 5. Multipart Upload Complete (Steps)

1. Validate all part numbers and etags from client.
2. Call ObjectStore.completeMultipartUpload(upload_id, key, parts).
3. Update files set object_key = key, size = sum(part_sizes), status = ready, updated_at = now() where file_id = ?
4. Delete upload_sessions row.
5. Return file metadata.

---

## 6. Hierarchy and Path

- List children: SELECT * FROM files WHERE parent_id = ? AND (owner_id = ? OR id IN (SELECT resource_id FROM shares WHERE shared_with = ?)) AND status != 'deleted' ORDER BY type, name LIMIT n.
- Path: either compute by walking parent_id from file to root, or store path_string and update on move (expensive for subtree).

---

## 7. Versioning

- On overwrite: copy current object_key to file_versions (new version_id); upload new content to new object_key; update files.object_key and files.version.
- Restore: copy version's object_key to current; or swap; create new version entry for current before restore.

---

## 8. Error Handling

- Duplicate name in same folder: 409 or auto-rename (name (1).pdf).
- Permission denied: 403.
- Upload part missing or wrong etag: 400 on complete.
- File not found or deleted: 404.
