---
name: session-kickoff
description: Start a focused session. Breaks work into discrete tasks and queues them one-at-a-time. Run at the start of every session.
disable-model-invocation: true
allowed-tools: Read, Glob, Grep, Bash
---

# Session Kickoff

## Step 1: Gather Context
- Read roadmap.md for pending items
- Run `git log --oneline -5` for recent work
- Run `git status` for uncommitted changes

## Step 2: Define Task Queue

Break the user's input (or roadmap items) into discrete tasks. Each task must be:
- Completable in one focused pass
- Independently committable
- Describable in one sentence

Present as:

```
## Session Plan — [date]

1. **[task]** — [one sentence] → `type/branch-name`
2. **[task]** — [one sentence] → `type/branch-name`
3. **[task]** — [one sentence] → `type/branch-name`

Ready to start #1?
```

## Step 3: Enforce One-at-a-Time

For each task:
1. State what we're doing
2. Plan briefly (files to touch, approach)
3. Implement
4. Verify (tests, lint)
5. Review (spawn reviewer agent)
6. Commit with proper message
7. Report: "✅ Task N complete. Moving to Task N+1: [desc]. Ready?"

Rules:
- Never start next task until current is committed
- If task is bigger than expected, suggest splitting
- If scope creeps ("while we're here..."), note it for later, stay focused
- User can say "skip to task N" or "add a task" at any time
