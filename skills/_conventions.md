# Skill Conventions

Shared patterns that all skills should follow. This is a reference doc, not a skill — it exists so skill authors (human or AI) stay consistent.

---

## Completion Status

Every skill ends by reporting one of:

- **DONE** — All steps completed successfully. Evidence provided.
- **DONE_WITH_CONCERNS** — Completed, but with issues to flag. List each concern.
- **BLOCKED** — Cannot proceed. State what's blocking and what was tried.
- **NEEDS_CONTEXT** — Missing information required to continue. State exactly what's needed.

## Escalation Protocol

It's always OK to stop and say "this is too hard" or "I'm not confident."

- If you've attempted a task 3 times without success, STOP and escalate.
- If you're uncertain about a security-sensitive change, STOP and escalate.
- If scope exceeds what you can verify, STOP and escalate.
- Bad work is worse than no work.

Format:
```
STATUS: BLOCKED | NEEDS_CONTEXT
REASON: [1-2 sentences]
ATTEMPTED: [what you tried]
RECOMMENDATION: [what the user should do next]
```

## AskUserQuestion Format

When asking Sameer a question during skill execution:

1. **Re-ground:** State the project, current branch, and what you're working on. (1-2 sentences)
2. **Simplify:** Explain the problem in plain English. No jargon, no internal function names. Say what it DOES, not what it's called.
3. **Recommend:** `RECOMMENDATION: Choose [X] because [one-line reason]`
4. **Options:** Lettered options: `A) ... B) ... C) ...`

Assume Sameer hasn't looked at this window in 20 minutes.

## Iron Laws

Collected from across skills. Each skill may have its own, but these are universal:

- **No fixes without root cause.** Understand why before changing what. (/investigate)
- **No ship claims without fresh test evidence.** Run the tests. Show the output. (/verify)
- **One feature, one commit, no scope creep.** Do the thing you set out to do. (/feature-flow)
- **No changes without measurement.** Baseline before you optimize.
- **Contract before code.** Define inputs, outputs, and schema before building.
- **Heuristic before model.** Start simple, add complexity only when the simple thing fails.

## Engineering Preferences

When reviewing or writing code:
- DRY but not at the cost of readability
- Well-tested: integration over unit, test behavior not implementation
- Engineered-enough: not over, not under
- Edge-case bias: handle the weird inputs
- Explicit over clever
- Minimal diff: smallest change that solves the problem
- Observability is not optional: if it breaks, can you tell?

## Browse Binary Setup

For skills that use the headless browser (`/qa`, `/design-review`, `/ux-audit`):

```bash
B=~/.claude/bin/browse
if [ ! -x "$B" ]; then echo "ERROR: browse binary not found at $B"; fi
```

For skills that use the design binary (`/design-review`, `/plan-design-review`):

```bash
D=~/.claude/bin/design
if [ -x "$D" ]; then echo "DESIGN_READY"; else echo "DESIGN_NOT_AVAILABLE"; fi
```
