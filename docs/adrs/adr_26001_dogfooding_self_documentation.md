---
id: 26001
title: "Dogfooding for Self-Documentation"
date: 2026-02-07
status: proposed
superseded_by: null
tags: [governance, documentation, workflow]
---

# ADR-26001: Dogfooding for Self-Documentation

## Date

2026-02-07

## Status

Proposed

## Context

As `vadocs` is a tool designed for managing documentation ecosystems, it is essential to determine the best practice for maintaining its own documentation. We need to ensure high documentation quality, consistency in metadata, and adherence to architectural standards while minimizing friction in the development workflow.

## Decision

We adopt a **dogfooding** strategy where `vadocs` is used to manage, validate, and fix its own documentation within this repository. 

This means that as new validation rules or fixing capabilities are added to the package, they should be applied to the project's own documentation suite. This ensures that the tool's features are continuously tested against real-world documentation needs, providing a tight feedback loop for development and ensuring the project's own documentation serves as a canonical example of the tool's capabilities.

## Consequences

### Positive

- **Atomic Changes:** Updates to features, documentation, and validation rules can be bundled in a single Pull Request.
- **Immediate Feedback:** Using the tool on its own repository is the most effective way to identify bugs and missing features during development.
- **CI/CD Integration:** Validation can be automated as a build step, ensuring the repository never contains invalid documentation.
- **No Version Lag:** The documentation is always validated by the version of the tool it describes, avoiding compatibility issues between separate repositories.

### Negative / Risks

- **Circular Dependency:** If a breaking change is introduced to the core validation logic, the tool may fail to validate its own documentation. **Mitigation**: Use stable releases for CI validation and editable installs for local development.
- **Local Setup Overhead:** Developers must install the package in editable mode to run self-validation. **Mitigation**: Document the setup process clearly in the [development guide](/docs/for_developers/04_development.md).

## Alternatives

- **Dedicated Documentation Repository:** Move documentation to a separate repository and install `vadocs` there. **Rejection Reason**: Increases overhead for cross-repo synchronization and creates versioning friction.
- **Manual Validation:** Rely on manual reviews to ensure documentation standards. **Rejection Reason**: Error-prone, does not scale, and fails to leverage the tool's primary purpose.

## References

- [Development Workflow](docs/for_developers/04_development.md)

## Participants

1. Vadim Rudakov
2. Gemini-3-Flash (aider architect)
