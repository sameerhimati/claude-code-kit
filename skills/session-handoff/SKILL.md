---
name: session-handoff
description: Write session handoff to session-handoff.md for the next session's /kickoff to read. Always updates the existing file, never creates a second one.
disable-model-invocation: true
allowed-tools: Bash, Read, Write, Edit, Glob, Grep
---

# Session Handoff

## Steps
1. Run `git log --oneline -10`
2. Run `git status`
3. Run `git branch --show-current`
4. Run build/test command if known
5. Read roadmap.md if it exists
6. Read existing session-handoff.md if it exists (to preserve history context)

## Write to session-handoff.md

**CRITICAL: If session-handoff.md already exists, OVERWRITE it. Do NOT create session-handoff-2.md or any variant. One file, always updated in place.**

Use the Write tool (or Edit tool if updating specific sections) to write `session-handoff.md` in the project root:

```markdown
# Session Handoff
> Last updated: [date and time]

## Completed This Session
- [x] [from git log — what was done this session]

## Current State
- **Branch:** [current branch]
- **Last commit:** [hash + message]
- **Build:** [passing/failing/unknown]
- **Uncommitted changes:** [yes/no — list files if yes]
- **Blockers:** [any blockers or "none"]

## Next Session Should
1. [highest priority next task]
2. [second priority]
3. [additional items from roadmap]

## Context to Remember
- [architectural decisions made and why]
- [gotchas discovered during this session]
- [things that almost broke or were tricky]
- [dependencies or external setup that was done]

## Start Command
`[specific command to resume work, e.g. "cd api && source venv/bin/activate && uvicorn app.main:app --reload"]`
```

## Rules
- Always write to `session-handoff.md` in the project root
- If the file exists, replace its contents entirely — do not append or create duplicates
- Be specific in "Next Session Should" — not "continue working" but "implement the /activities POST endpoint with Pydantic validation"
- "Context to Remember" is the most valuable section — capture things that would take time to re-discover
- Keep it concise. This is a handoff, not a diary.
