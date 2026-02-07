# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

vadocs is a validation engine for Documentation-as-Code workflows. It validates structure, enforces consistency, and auto-fixes common issues in documentation (particularly ADRs). This is a spoke project of the [AI Engineering Book](https://github.com/lefthand67/ai_engineering_book) — the parent project's ADRs in `architecture/adr/` are the primary target documents for vadocs validators.

Currently v0.1.0 — library-only, no CLI. v0.2.0 will add CLI subcommands and plugin discovery via entry points.

## Commands

```bash
# Run all tests
uv run pytest

# Run a single test file
uv run pytest tests/validators/test_adr.py

# Run a specific test
uv run pytest tests/validators/test_adr.py::TestAdrValidator::test_validate_missing_fields

# Run with coverage
uv run pytest --cov=src/vadocs

# Build package
uv build
```

No linter/formatter or CLI entry point is configured yet — library is used via direct imports only.

## Architecture

Three-step pipeline: **Extract → Analyze → Act**

1. **Extract (Parsing)** — `core/parsing.py`: Convert markdown files to `Document` dataclasses by parsing YAML frontmatter, extracting status, and section content via regex.
2. **Analyze (Validation)** — `validators/`: Run validator plugins against documents, returning `ValidationError` lists. Each validator has `supports(doc)` to filter by doc type and `validate(doc, config)`.
3. **Act (Fixing)** — `fixers/`: Run fixer plugins to auto-correct issues, returning `SyncResult`. Each fixer has `supports(doc)` and `fix(doc, config, dry_run)`.

### Key design decisions

- **Plugin contracts via ABCs**: `Validator` and `Fixer` base classes in `validators/base.py` and `fixers/base.py`. Also have Protocol versions for duck-typing.
- **Stateless plugins**: Validators and fixers receive all context through arguments — no internal state.
- **Configuration-driven**: All validation rules loaded from YAML config files via `config.load_config()`. Config includes required fields, allowed values, sync direction, status corrections.
- **Bi-directional sync**: `SyncFixer` can sync YAML→Markdown, Markdown→YAML, or auto-detect direction. Conflict detection when both sides differ.
- **Dry-run support**: All fixers accept `dry_run=True` to preview changes without modifying content.

### Source layout

```
src/vadocs/
├── __init__.py          # Public API (re-exports everything)
├── config.py            # YAML config loader
├── core/
│   ├── models.py        # Document, ValidationError, SyncField, SyncResult
│   └── parsing.py       # Frontmatter/section regex parsing
├── validators/
│   ├── base.py          # Validator ABC + Protocol
│   ├── adr.py           # ADR-specific validation rules
│   ├── frontmatter.py   # Generic frontmatter validation
│   └── myst_glossary.py # MyST {term} reference validation
└── fixers/
    ├── base.py          # Fixer ABC + Protocol
    ├── sync_fixer.py    # Bi-directional YAML↔Markdown sync
    └── adr_fixer.py     # ADR-specific auto-fixes
```

## Conventions

- **Save plans to `misc/plan/plan_<date_hash>.md`** before starting implementation. This preserves decision history across context switches.
- Use `uv run` for tests/scripts, `uv add` for dependencies — never raw pip.
- When editing `.md` files: these are MyST Markdown notebooks paired with Jupytext. Never convert `` ```{code-cell} `` blocks to standard fenced code blocks. Preserve MyST directive syntax exactly.
- Prefer `pathlib.Path` — never use `os` for path operations.
- Follow top-down design: main function / entry point at the top of the file.
- Use placeholders like `<IP_ADDRESS>` or `<DOMAIN>` in config files instead of real values.
- Atomic commits: one task per commit so it can be reverted as a whole unit.

## Testing Standards (from parent project)

- **TDD**: Write tests first, then implement (Red → Green → Refactor).
- **Test contracts, not implementation**: Verify exit codes, return types, side effects — not exact message strings.
- **Semantic assertions**: `assert len(errors) > 0` instead of exact counts when the count isn't the contract.
- **Document the contract**: Each test class should have a docstring explaining what contract it verifies.
- **Commit conventions**: Use conventional commits — `feat:`, `fix:`, `docs:`, `ci:`, `chore:`, `pr:` (promotional posts).
