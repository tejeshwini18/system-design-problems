# LLD: Pastebin

## APIs

```http
POST   /v1/pastes   Body: { content, expiry: "10m"|"1h"|"1d"|"never", custom_slug?, password? }
  Response: { slug, url, expires_at }

GET    /:slug   → raw content or HTML page (syntax highlight optional)
GET    /:slug/raw   → raw only
```

## Key Classes

```text
PasteService – create(content, expiry, customSlug?) → slug; get(slug) → content | 404
SlugGenerator – generate() → random 8-char; or base62(counter)
PasteRepository – save(slug, content, expiresAt); get(slug) → paste | null; deleteExpired() cron
```

## Expiry

- expiry "10m" → expires_at = now + 10 min. On get: if now > expires_at, return 404 and optionally delete. Cron: DELETE FROM pastes WHERE expires_at < NOW().

## Error Handling

- Duplicate custom_slug: 409. Content too large: 413. Invalid expiry: 400.
