# Validators

Modules: `vadocs.validators.base`, `vadocs.validators.adr`, `vadocs.validators.frontmatter`, `vadocs.validators.myst_glossary`

> For a conceptual overview, see [Developer Guide: Plugins](/docs/developer_guide/03_plugins.md).

Validators are stateless plugins that check a [Document](models.md) against configurable rules and return a list of [ValidationError](models.md) objects. Each validator implements `supports()` to filter by document type and `validate()` to perform checks.

---

## Base Classes

### class Validator

Abstract base class for validators. Subclass this to create a new validator.

Module: `vadocs.validators.base`

**Attributes:**

- **name** (`str`) — Unique identifier for this validator (used in config/CLI). Default: `"base"`.

**Abstract methods:**

```python
validate(document, config) → list[ValidationError]
```

- **document** (`Document`) — The document to validate.
- **config** (`dict`) — Configuration dictionary (repo-specific settings).
- Returns a list of `ValidationError` objects. Empty list if the document is valid.

```python
supports(document) → bool
```

- **document** (`Document`) — The document to check.
- Returns `True` if this validator can validate this document type.

---

### class ValidatorProtocol

Protocol for duck-typed validators. Use this for type checking when you don't want to subclass the ABC.

Module: `vadocs.validators.base`

**Required attributes and methods:** Same as `Validator` — `name`, `validate()`, `supports()`.

---

## Built-in Validators

### class AdrValidator

Validates Architecture Decision Records against a full set of configurable rules.

Module: `vadocs.validators.adr`

- **name:** `"adr"`
- **supports:** Documents with `doc_type="adr"` or filenames matching `adr_*.md`.

**Checks performed:**

| Check | Config key | Error type |
|---|---|---|
| Required frontmatter fields present | `required_fields` | `missing_field` |
| Status is a valid value | `statuses` | `invalid_status` |
| Date matches format pattern | `date_format` | `invalid_date` |
| Tags are from allowed list | `tags` | `invalid_tag` |
| Tags list is not empty | — | `empty_tags` |
| Required markdown sections present | `required_sections` | `missing_section` |
| Title matches between header and frontmatter | — | `title_mismatch` |

**ADR number extraction** (used as `identifier` in errors): tries frontmatter `id` field, then filename pattern `adr_XXXXX_*.md`, then header pattern `# ADR-XXXXX:`. Falls back to `0`.

---

### class FrontmatterValidator

Generic validator for YAML frontmatter in any document type.

Module: `vadocs.validators.frontmatter`

- **name:** `"frontmatter"`
- **supports:** Any document with YAML frontmatter (parsed or parseable from content).

**Checks performed:**

| Check | Config key | Error type |
|---|---|---|
| Frontmatter exists (if required fields configured) | `required_fields` | `missing_frontmatter` |
| Required fields present | `required_fields` | `missing_field` |
| Field values in allowed lists | `allowed_values` | `invalid_value` |

The `allowed_values` config key is a dict mapping field names to lists of allowed values. Works with both scalar and list-typed fields (e.g., tags).

---

### class MystGlossaryValidator

Base validator for MyST glossary term references. Subclass this for specific term reference patterns.

Module: `vadocs.validators.myst_glossary`

- **name:** `"myst_glossary"`
- **supports:** Any `.md` file.

This is an abstract class — you must override `get_patterns()`:

```python
get_patterns(config) → list[dict]
```

Each pattern dict contains:

- **pattern** (`str`) — Regex pattern string.
- **make_suggestion** (`Callable(match) → str`) — Returns the corrected text.
- **error_type** (`str`, optional) — Error type string. Default: `"broken_term_reference"`.
- **get_identifier** (`Callable(match) → identifier`, optional) — Extracts identifier from match.

---

### class AdrTermValidator

Detects broken `{term}\`ADR XXXXX\`` references that should use a hyphen separator.

Module: `vadocs.validators.myst_glossary`

- **name:** `"myst"`
- **supports:** Any `.md` file (inherited from `MystGlossaryValidator`).

**Checks performed:**

| Check | Config key | Error type |
|---|---|---|
| ADR term references use correct separator | `term_reference.separator`, `term_reference.broken_pattern` | `broken_term_reference` |

**Defaults:**

- `broken_pattern`: `\{term\}\`ADR (\d+)\`` (matches space separator)
- `separator`: `"-"` (corrects to hyphen)

**Config example:**

```yaml
term_reference:
  separator: "-"
  broken_pattern: "\\{term\\}`ADR (\\d+)`"
```
