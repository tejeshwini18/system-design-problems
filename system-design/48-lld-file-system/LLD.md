# LLD: File System (In-Memory / Hierarchical)

## 1. Requirements

- **Hierarchy:** Directories and files; root "/"; path like "/home/user/file.txt".
- **Directory:** Can contain files and subdirectories; has name, parent, children.
- **File:** Has name, parent directory, content (string or blob), size.
- **Operations:** createFile(path, content), createDirectory(path), getFile(path), list(path), delete(path), move(src, dest).
- **No cycles:** Parent-child is tree.
- **Optional:** Permissions, symlinks, file size limits.

---

## 2. Core Classes (Composite Pattern)

```text
FileSystemEntity (abstract) – name, parent (Directory), path (computed or cached)
  getPath() → full path string
  getSize() → long (file: content length; directory: sum of children sizes or 0)

File extends FileSystemEntity – content: byte[] or String; getSize() = content.length

Directory extends FileSystemEntity – children: List<FileSystemEntity>
  addChild(entity), removeChild(name), getChild(name), list() → list of names or entities
  getSize() → sum of children getSize() or 0

FileSystem – root: Directory
  createFile(path, content): resolve parent directory from path; create File; parent.addChild(file)
  createDirectory(path): resolve parent; create Directory; parent.addChild(dir)
  get(path): resolve path to entity (file or directory)
  list(path): get(path) must be Directory; return directory.list()
  delete(path): get(path); parent.removeChild(entity.name)
  move(src, dest): get(src); get parent of dest; remove from old parent; add to new parent (rename if name in path)
```

---

## 3. Path Resolution

- **Split path:** "/home/user/doc" → ["", "home", "user", "doc"]; ignore empty first.
- **Traverse:** Start at root; for each segment "home", "user", "doc": current = current.getChild(segment); if null and not last segment, fail (parent not found); if last segment, that is target.
- **Create:** For "/a/b/c", get or create "/a", then "/a/b", then create file "c" under "/a/b".

---

## 4. Design Patterns

- **Composite:** FileSystemEntity with File and Directory; Directory holds List<FileSystemEntity>; same interface (getPath, getSize); list() only on Directory.
- **Optional:** **Proxy** for lazy-loading content of large files; **Visitor** for operations over whole tree (e.g. total size, search).

---

## 5. Key Methods

```text
FileSystem.createFile(path, content)
FileSystem.createDirectory(path)
FileSystem.get(path) → File | Directory | null
FileSystem.list(path) → List<String>
FileSystem.delete(path)
FileSystem.move(srcPath, destPath)
Directory.addChild(entity), getChild(name), list()
File.getContent(), setContent()
```

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

