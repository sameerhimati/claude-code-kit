---
name: session-handoff
description: Write session handoff to session-handoff.md for the next session's /kickoff to read. Always updates the existing file, never creates a second one.
disable-model-invocation: false
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
> **Last session:** [YYYY-MM-DD] — [short topic, e.g. "Phase 2A core models"]
> **Current frame:** [ONE sentence describing what we're actually building. Rewritten ONLY when the strategic framing changes — most sessions inherit the frame from the previous handoff unchanged. A future-me reading this cold should know within 5 seconds what we're building and why.]

## Completed This Session
- [x] [what was actually done — include the key decision or insight, not just the diff. Git log has the diff; this section has the meaning.]

## Current State
- **Branch:** [current branch]
- **Last commit:** [hash + message]
- **Tests:** [N passing / failing / added this session]
- **Build:** [passing/failing/unknown]
- **Uncommitted changes:** [yes/no — list files if yes]

## Next Action
[ONE thing. The next concrete action to take. Not a menu, not a 6-task breakdown. If the work has more structure, point at the plan doc (e.g. "see docs/block2-plan.md §3") instead of re-listing. Optional "After that:" line for immediate follow-up context, but the primary focus is singular.]

## Blockers
[External things waiting on input: legal consults, partnership asks, customer meetings, handoffs from others. "None" is a valid answer. Do not pad.]

## Context to Remember
[Session-specific things that aren't in CLAUDE.md and will decay over time:
- Mid-work decisions not yet durable
- Emotional state if relevant to next session judgment
- Gotchas discovered this session
- Things that would take time to re-discover

This section SHRINKS over time. When something becomes durable, move it to CLAUDE.md. When it becomes irrelevant, delete it.]

## Start Command
```bash
[specific command to resume work]
```
```

## Rules
- Always write to `session-handoff.md` in the project root
- If the file exists, replace its contents entirely — do not append or create duplicates
- **Current frame is load-bearing.** A future-me reading the handoff cold should know within 5 seconds what we're building. Inherit it unchanged from the previous handoff unless a strategic reframe happened in this session.
- **Next Action is singular, not a list.** If there's more structure, point at the plan doc. This forces focus and avoids rework when plans change.
- **Blockers are honest.** If nothing is blocking, write "none." If something is waiting on external input, say what specifically. Don't pad.
- **Context to Remember decays.** This is the section that should shrink over time. Durable items graduate to CLAUDE.md; irrelevant items get deleted. If it's growing every session, something's wrong.
- **Architectural decisions live in CLAUDE.md, not here.** Don't duplicate them across handoffs — they get re-debated that way.
- Keep it concise. Target 60–120 lines. This is a handoff, not a diary.
