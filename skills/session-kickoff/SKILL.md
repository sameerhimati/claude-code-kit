---
name: session-kickoff
description: Start a focused session. Reads session-handoff.md for context from last session, then builds a task queue.
disable-model-invocation: true
allowed-tools: Read, Glob, Grep, Bash
---

# Session Kickoff

## Step 1: Gather Context

Read these in order (skip if they don't exist):
1. **session-handoff.md** — previous session's handoff (highest priority, has immediate context)
2. **roadmap.md** — project roadmap for pending items
3. Run `git log --oneline -5` for recent work
4. Run `git status` for uncommitted changes

If session-handoff.md exists, summarize what was done last time and what was recommended next. This is the starting point for planning.

## Step 2: Define Task Queue

Break the user's input (or handoff recommendations + roadmap items) into discrete tasks. Each task must be:
- Completable in one focused pass
- Independently committable
- Describable in one sentence

Present as:

```
## Session Plan — [date]

### Context from Last Session
[1-2 sentences from session-handoff.md, or "Fresh start — no prior handoff"]

### Tasks
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
