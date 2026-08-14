---
name: code-standards
description: Coding standards and patterns. Loaded automatically when writing or reviewing code.
disable-model-invocation: false
---

# Code Standards

## Principles
- Explicit over implicit
- Composition over inheritance
- Comments explain WHY, not WHAT
- Handle errors at the appropriate level
- One function, one job
- Prefer early returns to reduce nesting

## Naming
- Variables: descriptive, camelCase (JS/TS) or snake_case (Python)
- Functions: verb-first (getUserById, calculate_total)
- Booleans: is/has/should prefix (isActive, hasPermission)
- Constants: UPPER_SNAKE_CASE
- Files: kebab-case for non-components, PascalCase for components

## Error Handling
- Never use empty catch blocks
- Use typed errors where possible
- Log errors with context (operation, relevant IDs)
- User-facing errors should be helpful, not technical

## File Organization
- Group by feature, not by type
- Keep files under 300 lines
- Index files only for public API re-exports

## Testing
- Test behavior, not implementation
- Name tests: "should return 404 when user not found"
- One assertion concept per test
