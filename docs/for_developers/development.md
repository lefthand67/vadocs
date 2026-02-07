# Development Workflow

## Packaging
The project uses `hatchling` as the build backend, configured in `pyproject.toml`. 
- Dependencies: `pyyaml`, `jupytext`, `pytest`.
- Python Requirement: `>=3.13`.

## Testing
Tests are located in `tests/` and use `pytest`.
- **Fixtures:** Common test data (sample Markdown with/without frontmatter) is defined in `tests/conftest.py`.
- **Core Tests:** `tests/core/` contains tests for models and parsing logic.

To run the test suite:
```bash
pytest
```

## Configuration
Validators and fixers are configured via YAML files loaded by `vadocs.config.load_config`. Future versions will support `pyproject.toml` configuration.
