# Core Data Models

The system uses Python dataclasses (defined in `src/vadocs/core/models.py`) to maintain state and communicate results.

## Document
The primary data structure representing a documentation file.
- `path`: `Path` to the source file.
- `content`: Raw string content of the file.
- `frontmatter`: Dictionary of parsed YAML data (or `None`).
- `doc_type`: String identifier (e.g., "adr") used to determine applicable plugins.

## ValidationError
Represents a single rule violation.
- `identifier`: Context-specific ID (e.g., ADR number or filename).
- `error_type`: Category (e.g., "missing_field", "invalid_status").
- `message`: Human-readable description.

## SyncField
Represents a field that exists in both YAML and Markdown.
- `is_synced`: Property that returns `True` if values match or if only one source has data.

## SyncResult
The output of a `Fixer` operation.
- `modified`: Boolean indicating if the document was changed.
- `changes`: List of strings describing modifications.
- `errors`: List of strings describing failures.
