---
name: session-handoff
description: Generate a session handoff prompt for the next Claude Code session.
disable-model-invocation: true
allowed-tools: Bash, Read, Glob, Grep
---

# Session Handoff

## Steps
1. Run `git log --oneline -10`
2. Run `git status`
3. Run `git branch --show-current`
4. Run build/test command if known
5. Read roadmap.md if it exists

## Generate

```
## Session Handoff — [Project Name]
**Date:** [today]
**Branch:** [current]

### Completed This Session
- [x] [from git log]

### Current State
- Last commit: [message]
- Build: [passing/failing]
- Uncommitted changes: [yes/no]
- Blockers: [any]

### Next Session Should
1. [from roadmap]
2. [next task]

### Context to Remember
- [decisions made]
- [gotchas found]

### Start Command
`[specific command to begin]`
```
