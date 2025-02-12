
# Version Labels

## Overview

Version labels are identifiers used to mark specific points in a project's development timeline. They help track releases, identify bugs, and manage software versions effectively.

## Common Version Label Types

### Release Labels

- **Semantic Version (e.g., v2.1.3)**
  - MAJOR version for incompatible API changes
  - MINOR version for backwards-compatible features
  - PATCH version for backwards-compatible fixes

- **Pre-release Labels**
  - alpha (v2.0.0-alpha)
  - beta (v2.0.0-beta)
  - release candidate (v2.0.0-rc.1)

### Development Labels

- **Build Numbers** (e.g., build.123)
  - Unique identifier for each build
  - Often used in continuous integration

- **Commit Hashes**
  - Full hash (e.g., 8f4e821b912)
  - Short hash (e.g., 8f4e821)

- **Date-Based Versions** (e.g., 20231015)
  - Use YYYYMMDD format in versioning
    - enables ASCII-based sorting
  - Helps in tracking changes on a daily basis

## Best Practices

1. **Consistent Format**
   - Use standard prefixes (v1.0.0 vs 1.0.0)
   - Follow project conventions
   - Document labeling scheme

2. **Clear Progression**
   - Ensure version numbers increase logically
   - Use pre-release labels for testing
   - Mark breaking changes clearly

3. **Tag Management**
   - Create tags at significant points
   - Document changes in release notes
   - Keep tags synchronized across branches

## Tools and Commands

- Use `git tag` to create version labels
- Use release management tools when available
- Automate version bumping when possible

## Further Reading

- [Semantic Versioning 2.0.0](https://semver.org/)
- [Git Basics - Tagging](https://git-scm.com/book/en/v2/Git-Basics-Tagging)
