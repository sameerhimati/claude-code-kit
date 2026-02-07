---
name: research
description: Structured parallel research. Spawns subagents for web, codebase, and community sources.
disable-model-invocation: true
allowed-tools: Task, WebSearch, WebFetch, Grep, Glob, Read
---

# Research: $ARGUMENTS

## Step 1: Launch Parallel Subagents

1. **Web Docs Agent** — official docs, best practices, GitHub issues
2. **Codebase Agent** — existing patterns, related code, dependencies
3. **Community Agent** — blog posts, Stack Overflow, real-world experiences

## Step 2: Synthesize

```
## Research: [topic]

### Key Findings
[Most important takeaways]

### Recommended Approach
[What to do, based on evidence]

### Alternatives Considered
[Other approaches and tradeoffs]

### Pitfalls to Avoid
[Common mistakes]

### Sources
[Key references]
```
