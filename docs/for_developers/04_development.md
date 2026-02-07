# Development Workflow


## Environment & Packaging


The project uses `uv` for dependency management and `hatchling` as the build backend, configured in `pyproject.toml`.

- **Python Requirement:** `>=3.13`.
- **Core Dependencies:** `pyyaml`, `jupytext`, `myst`.
- **Dev Dependencies:** `pytest`, `pytest-cov`.

To set up the development environment:
```bash
uv sync
```


## Testing


Tests are located in `tests/` and use `pytest`.
- **Fixtures:** Common test data (sample Markdown with/without frontmatter) is defined in `tests/conftest.py`.
- **Core Tests:** `tests/core/` contains tests for models and parsing logic.

To run the test suite with coverage reporting:
```bash
uv run pytest tests/ --cov=. --cov-report=term-missing -q
```


## Dogfooding (Self-Validation)


`vadocs` is used to maintain its own documentation. This "dogfooding" approach ensures the tool remains functional and identifies missing features during development. For more details on this decision, see {term}`ADR-26030`.

- **Validation:** The tool's validation engine is applied to all documentation in the `docs/` directory to ensure consistency and adherence to standards.
- **Fixing:** Automated fixers are used to maintain the documentation suite, ensuring that the project's own docs serve as a canonical example of the tool's capabilities.
- **Editable Install:** For local development and self-validation, install the package in editable mode:
  ```bash
  uv pip install -e .
  ```


## Configuration


Validators and fixers are currently configured via external YAML files loaded by `vadocs.config.load_config`. Future versions will support `pyproject.toml` configuration.


## Roadmap


Following the successful v0.1.0 Proof of Concept, the planned evolution is:
- **v0.2.0:** CLI interface and `pyproject.toml` `[tool.vadocs]` config loading.
- **v0.3.0:** Index sync validation (ensuring ADR indexes match file headers).
- **v0.4.0:** Additional validators (broken links, jupytext consistency).
- **v1.0.0:** PyPI release and stable API.
