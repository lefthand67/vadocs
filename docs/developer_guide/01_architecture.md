# Architecture Overview

`vadocs` is a modular Python package designed to validate and synchronize documentation files (primarily Markdown with YAML frontmatter). It uses a "src" layout to ensure clean packaging and testing.

## The Pipeline
The package follows a three-step workflow:

1.  **Extract (Parsing):** Files are read and converted into `Document` objects. The `vadocs.core.parsing` module uses regular expressions to separate YAML frontmatter from Markdown content and identify specific sections (like "Status").
2.  **Analyze (Validation):** The `Document` is passed through a series of `Validator` plugins. Each validator checks for specific rules (e.g., required fields, valid statuses, or correct link formatting) and returns a list of `ValidationError` objects.
3.  **Act (Fixing):** If requested, `Fixer` plugins attempt to resolve issues. This includes updating frontmatter to match content (or vice versa) and correcting common formatting errors.

## Project Structure
- `src/vadocs/core/`: Core data structures (`models.py`) and parsing logic (`parsing.py`).
- `src/vadocs/validators/`: Plugin-based validation rules.
- `src/vadocs/fixers/`: Logic for automated document corrections and bi-directional sync.
- `src/vadocs/config.py`: Minimal YAML configuration loader.
