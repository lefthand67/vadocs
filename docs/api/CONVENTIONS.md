# API Documentation Conventions

Instructions for LLM agents updating vadocs API reference pages.

## Format Reference

The API docs use a psycopg3-inspired, Markdown-native format. Follow these patterns exactly when adding or updating documentation.

### Psycopg3 Reference Pages

When in doubt about formatting, explore these psycopg3 docs as the upstream inspiration:

- [API index](https://www.psycopg.org/psycopg3/docs/api/index.html) — how they organize the top-level API landing page
- [Connection classes](https://www.psycopg.org/psycopg3/docs/api/connections.html) — class headers, method signatures, parameter lists, cross-refs
- [The psycopg module](https://www.psycopg.org/psycopg3/docs/api/module.html) — module-level functions and constants
- [sql module](https://www.psycopg.org/psycopg3/docs/api/sql.html) — enum documentation, class hierarchies
- [Other top-level objects](https://www.psycopg.org/psycopg3/docs/api/objects.html) — dataclass-style objects, properties

Key patterns to adapt from psycopg3: class header with module path, signatures with `:param:` → bulleted parameter lists, `:rtype:` → `→` return type in signature, `.. note::` → `> **Note:**` blockquotes, semantic method grouping under H3 headers.

### Class Header

```markdown
### class ClassName

Description of the class in 1-2 sentences.

Module: `vadocs.module.submodule`
```

### Constructor / Function Signature

Use a Python code block with `→` for return type:

```markdown
```python
function_name(param1, param2, keyword=default) → ReturnType
`` `
```

### Parameters

Bulleted list with bold name, type in parens, em-dash, description:

```markdown
**Parameters:**

- **param_name** (`type`) — Description.
- **other_param** (`type`, optional) — Description. Default: `value`.
```

### Return Values

Inline after signature or as a separate block:

```markdown
**Returns:** Description of what is returned, or `None` if condition.
```

### Properties

```markdown
**Properties:**

- **prop_name** (`type`) — Description.
```

### Tables for Checks / Fixes

Use tables to summarize what a validator checks or what a fixer fixes:

```markdown
| Check | Config key | Error type |
|---|---|---|
| Description of check | `config_key` | `error_type_string` |
```

### Admonitions

Use blockquote with bold prefix:

```markdown
> **Note:** Important information here.
```

### Cross-References

- **API → Developer Guide** (near top of each page):
  ```markdown
  > For a conceptual overview, see [Developer Guide: Section Name](/docs/developer_guide/0N_page.md).
  ```
- **Developer Guide → API** (added to existing guide pages):
  ```markdown
  > See also: [API Reference: Page Name](/docs/api/page.md) for complete signatures and parameter details.
  ```

## File Organization

| File | Contents | Source modules |
|---|---|---|
| `index.md` | Package structure table, import reference, version | `vadocs/__init__.py` |
| `models.md` | Dataclasses: Document, ValidationError, SyncField, SyncResult | `vadocs/core/models.py` |
| `parsing.md` | Constants + functions: parse_frontmatter, extract_status, extract_section_content | `vadocs/core/parsing.py` |
| `validators.md` | ABC, Protocol, all built-in validators | `vadocs/validators/base.py`, `adr.py`, `frontmatter.py`, `myst_glossary.py` |
| `fixers.md` | ABC, Protocol, SyncDirection enum, all built-in fixers | `vadocs/fixers/base.py`, `sync_fixer.py`, `adr_fixer.py` |
| `configuration.md` | load_config function + config key reference | `vadocs/config.py`, `docs/adrs/adr_config.yaml` |

## Update Checklist

When updating API documentation:

1. **Read the source code** — signatures, types, docstrings, and default values are the source of truth.
2. **Check `__init__.py`** — every symbol in `__all__` must appear in at least one API page.
3. **Update `index.md`** — if adding new symbols, add them to the package structure table and import reference.
4. **Preserve MyST syntax** — never convert `{code-cell}` blocks to standard fenced code blocks in any `.md` files.
5. **Cross-reference both directions** — API pages link to developer guide; developer guide pages link back to API.
6. **Balance prose and signatures** — aim for ~40% explanatory text, ~60% signatures/tables/code.
7. **Keep error_type values accurate** — these are the programmatic strings used in code, not paraphrases.
8. **Document config keys per-plugin** — show which config keys each validator/fixer reads from.
9. **Dry-run contract** — always document how fixers behave with `dry_run=True`.
