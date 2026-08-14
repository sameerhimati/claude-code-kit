---
name: research
description: Use when investigating a technology, library, approach, or decision that requires evidence from multiple sources. Spawns parallel subagents for web docs, codebase analysis, and community experiences. Invoke with /research or triggered when deep investigation is needed before implementation.
disable-model-invocation: false
allowed-tools: Task, WebSearch, WebFetch, Grep, Glob, Read
---

# Research: $ARGUMENTS

## Step 0: Determine Depth

- **Quick** (default for small decisions): 1 agent, 5-min timebox, key findings only
- **Deep** (for architectural decisions, new tech adoption): 3 parallel agents, thorough synthesis
- If the user doesn't specify, infer from context. "Should I use Redis or in-memory cache?" = quick. "What's the best approach for real-time collaboration?" = deep.

## Step 1: Launch Parallel Subagents

1. **Web Docs Agent** — official docs, best practices, GitHub issues, changelogs
2. **Codebase Agent** — existing patterns, related code, dependencies, how similar things are done here
3. **Community Agent** — blog posts, Stack Overflow, X threads, real-world experiences (especially failure stories)

For quick research: combine all three into one agent.

## Step 2: Synthesize

```
## Research: [topic]

### Key Findings
[Most important takeaways — lead with the answer]

### Recommended Approach
[What to do, based on evidence. Be opinionated.]

### Alternatives Considered
[Other approaches and why they were rejected]

### Pitfalls to Avoid
[Common mistakes, failure modes, things that look good but aren't]

### Sources
[Key references with links]
```

## Step 3: Save to Vault (if insight is durable)

If the finding is a lasting insight (not just "use library X for this task"), save it to the knowledge vault:
- Cross-project pattern → `ideas/`
- Technology learning → `research/`
- Project-specific decision → `projects/<project>/`

## Completion Status

```
STATUS: DONE — Research complete. Recommendation: [one line].
STATUS: DONE_WITH_CONCERNS — Findings inconclusive. [What's uncertain and why.]
STATUS: NEEDS_CONTEXT — Can't research without knowing [specific missing info].
```

## Anti-Patterns
- BAD: Researching for 30 minutes when the answer is in the project's own docs. (Check codebase first.)
- BAD: Recommending the most popular option without checking if it fits. (Community hype ≠ right for this project.)
- BAD: "There are several options, each with tradeoffs." (Be opinionated. Recommend ONE approach.)
