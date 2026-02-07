# Extending vadocs: Validators and Fixers

`vadocs` uses a plugin architecture based on Abstract Base Classes (ABC) and Protocols for "duck typing" support.

## Creating a Validator
Subclass `vadocs.validators.base.Validator`:
1.  Implement `supports(document)`: Return `True` if the validator applies to this document.
2.  Implement `validate(document, config)`: Return a list of `ValidationError` objects.

## Creating a Fixer
Subclass `vadocs.fixers.base.Fixer`:
1.  Implement `supports(document)`: Return `True` if the fixer applies.
2.  Implement `fix(document, config, dry_run)`: Return a `SyncResult`. 
    - **Important:** Always respect the `dry_run` flag. Do not modify files or state if `dry_run` is `True`.

## Bi-directional Sync
The `SyncFixer` handles synchronization between YAML frontmatter and Markdown sections. It uses `SyncDirection` (YAML_TO_MD, MD_TO_YAML, or AUTO) to resolve conflicts.
