# Fixers

Modules: `vadocs.fixers.base`, `vadocs.fixers.sync_fixer`, `vadocs.fixers.adr_fixer`

> For a conceptual overview, see [Developer Guide: Plugins](/docs/developer_guide/03_plugins.md).

Fixers are stateless plugins that automatically correct documentation issues. They receive a [Document](models.md) and configuration, and return a [SyncResult](models.md) indicating what changes were made. All fixers support a `dry_run` mode that reports changes without writing files.

---

## Base Classes

### class Fixer

Abstract base class for fixers. Subclass this to create a new fixer.

Module: `vadocs.fixers.base`

**Attributes:**

- **name** (`str`) — Unique identifier for this fixer (used in config/CLI). Default: `"base"`.

**Abstract methods:**

```python
fix(document, config, dry_run=False) → SyncResult
```

- **document** (`Document`) — The document to fix.
- **config** (`dict`) — Configuration dictionary (repo-specific settings).
- **dry_run** (`bool`) — If `True`, report what would change without modifying files. Default: `False`.
- Returns a `SyncResult`. When `dry_run=True`, `modified` is always `False` but `changes` still lists what would happen.

```python
supports(document) → bool
```

- **document** (`Document`) — The document to check.
- Returns `True` if this fixer can fix this document type.

**Dry-run contract:** When `dry_run=True`, fixers must not write to the filesystem. `SyncResult.modified` must be `False`. `SyncResult.changes` should still describe what would be changed.

---

### class FixerProtocol

Protocol for duck-typed fixers. Use this for type checking when you don't want to subclass the ABC.

Module: `vadocs.fixers.base`

**Required attributes and methods:** Same as `Fixer` — `name`, `fix()`, `supports()`.

---

## Enums

### class SyncDirection

Direction for bi-directional sync between YAML frontmatter and markdown content.

Module: `vadocs.fixers.sync_fixer`

| Value | Description |
|---|---|
| `SyncDirection.AUTO` (`"auto"`) | Determine direction automatically. Works when only one source has data. Reports an error if both sources have conflicting values. |
| `SyncDirection.YAML_TO_MD` (`"yaml_to_md"`) | Use YAML frontmatter as source of truth. |
| `SyncDirection.MD_TO_YAML` (`"md_to_yaml"`) | Use markdown content as source of truth. |

---

## Built-in Fixers

### class SyncFixer

Bi-directional sync fixer for YAML frontmatter and markdown content. Synchronizes fields that exist in both locations (title, status, date).

Module: `vadocs.fixers.sync_fixer`

- **name:** `"sync"`
- **supports:** Any document with YAML frontmatter.

**Config keys:**

| Key | Type | Default | Description |
|---|---|---|---|
| `sync_direction` | `str` | `"auto"` | One of `"auto"`, `"yaml_to_md"`, `"md_to_yaml"` |
| `sync_fields` | `list[str]` | `["title", "status", "date"]` | Field names to synchronize |

**Sync behavior:**

- Fields where only one source has a value are synced in the available direction (regardless of `sync_direction` setting in `auto` mode).
- Fields where both sources differ require an explicit direction. In `auto` mode, these produce an error in `SyncResult.errors`.
- Title is extracted from the `# ADR-XXXXX: Title` header pattern.
- Status is extracted from the `## Status` section (first word, lowercased).
- Date is extracted from the `## Date` section (`YYYY-MM-DD` pattern).

**Change descriptions** (in `SyncResult.changes`):

- `"Updated markdown title to '...'"` / `"Updated YAML title to '...'"`
- `"Updated markdown status to '...'"` / `"Updated YAML status to '...'"`
- `"Updated markdown date to '...'"` / `"Updated YAML date to '...'"`

---

### class AdrFixer

Fixes common ADR issues: invalid statuses and title mismatches between header and frontmatter.

Module: `vadocs.fixers.adr_fixer`

- **name:** `"adr"`
- **supports:** Documents with `doc_type="adr"` or filenames matching `adr_*.md`.

**Fixes performed:**

| Fix | Config keys | Change description |
|---|---|---|
| Invalid status correction | `statuses`, `status_corrections`, `default_status` | `"Fixed status: '...' -> '...'"` |
| Title mismatch (frontmatter updated to match header) | — | `"Fixed title mismatch: '...' -> '...'"` |

**Status correction logic:** Looks up the current status in the `status_corrections` mapping. If no mapping found, falls back to `default_status` (default: `"proposed"`). Only applies the fix if the resulting status is in the `statuses` list.

**Title fix direction:** Always updates frontmatter to match the markdown header (header is source of truth).
