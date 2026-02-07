---
name: techdebt-scanner
description: Scan codebase for technical debt. Run at end of every session.
disable-model-invocation: true
allowed-tools: Bash, Read, Glob, Grep
---

# Tech Debt Scanner

## Scan For
1. **Duplicated code** — near-duplicate functions or blocks
2. **Unused imports/variables** — dead code
3. **TODO/FIXME/HACK comments** — list with file locations
4. **Inconsistent patterns** — mixed approaches to same problem
5. **Large files** — over 300 lines, should split
6. **Missing error handling** — empty catches, unhandled promises
7. **Dependency issues** — unused deps in package.json/requirements.txt

## Output

```
## Tech Debt Report — [date]

### 🔴 High Priority
- [finding with file:line]

### 🟡 Medium Priority
- [finding with file:line]

### 💡 Low Priority
- [finding with file:line]

### Top Refactor Recommendation
[Single highest-impact change that addresses the most debt]
```
