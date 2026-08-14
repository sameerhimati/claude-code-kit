---
name: feature-flow
description: Use when building a new feature, adding functionality, or implementing a user story. Chains plan → implement → verify → review → commit. One feature = one commit. Invoke with /feature-flow or auto-triggered when user describes a feature to build.
disable-model-invocation: false
allowed-tools: Task, Bash, Read, Write, Edit, Glob, Grep
---

# Feature: $ARGUMENTS

## Step 1: Scope Check
- Is this ONE feature or multiple? If multiple, stop and split.
- Can this be one commit message? If not, it's too big.

## Step 2: Plan
- Read existing code in the affected area first
- Which files change? New files needed?
- Data model changes?
- Edge cases?
- What could break?

Present plan. Wait for confirmation before proceeding.

## Step 3: Implement
- Follow existing codebase patterns (check code-standards)
- Don't touch unrelated code — note it for later
- Comments only for non-obvious logic
- Write tests alongside implementation, not after

## Step 4: Verify
Invoke /verify. This runs:
- Lint and type-check
- Tests (including new ones for this feature)
- Build
- Server smoke test

Do NOT skip this step. Do NOT move to review until verify passes.

## Step 5: Review (report) → Apply
Invoke /review in REPORT-ONLY mode (no edits). Read the actionable findings, then apply the eligible ones in the working tree.
- **Done criterion (checkable):** every actionable finding is either fixed in this commit or recorded as a residual (Step 5b). None left dangling.

## Step 5b: File residuals
Any finding not fixed in this feature (out of scope / needs a decision / flaky) must become durable before finishing:
- Append to the handoff summary (Step 7): severity, file:line, title, why deferred.
- If the repo tracks follow-ups (TODO.md, docs/residuals/), append there too and stage separately from the feature commit.
- **Iron rule:** a residual is "filed" only when written somewhere durable — not merely mentioned in chat.

## Step 6: Commit
- Stage ONLY files for this feature
- Commit message per commit-conventions
- Report: feature complete, commit hash, files changed, test status
- Push only on explicit user request — never auto-push.

## Step 6b: CI watch (only after an AUTHORIZED push)
Run ONLY if the commit was pushed at the user's request AND an open PR exists. Check: `gh pr view --json number,state` — skip silently if no PR or `gh` is unavailable.
For up to 3 fix iterations:
- `gh pr checks --watch`
- On failure: pull logs (`gh run view <run-id> --log-failed`), find the root cause, fix.
- ASK before pushing each fix — the push gate stays on.
Never fake green. STOP after 3 attempts; if still red, record the remaining failures as residuals (Step 5b) and report BLOCKED.

## Step 7: Handoff Summary
Output a structured summary for session continuity:
```
### Feature Complete: [name]
- **Commit:** [hash]
- **Files changed:** [count] ([list key files])
- **Tests:** [count passing]
- **What it does:** [one sentence]
- **What to watch:** [any risks or follow-ups]
```

## Step 8: Compound (write learnings back)
If the feature surfaced a durable, reusable lesson, PROPOSE adding it to the narrowest home:
- project-specific → project CLAUDE.md
- cross-project → `~/.claude/rules/` or a skill
- one-off → handoff only (don't persist).

Never auto-write to global `~/.claude` — ask first. Apply the deletion test before adding any line; don't let learnings sediment into CLAUDE.md.

## IRON LAW
**ONE FEATURE, ONE COMMIT, NO SCOPE CREEP.** If you can't describe the change in one commit message, it's too big. Split it. If you notice something unrelated, note it and move on.

## Completion Status

End every feature-flow with:
```
STATUS: DONE — Feature shipped, verified, reviewed, committed.
STATUS: DONE_WITH_CONCERNS — Shipped, but [specific concern or follow-up needed].
STATUS: BLOCKED — Cannot complete. [What's blocking and what was tried.]
STATUS: NEEDS_CONTEXT — Missing requirements. [What needs clarification.]
```

## Anti-Patterns
- BAD: "While I'm in this file, let me also fix that unrelated thing." (Note it. Move on. Don't pollute the commit.)
- BAD: Committing unrelated changes together. (One feature = one commit. Always.)
- BAD: "Review isn't needed for this small change." (Small changes cause big bugs. Review everything.)
- BAD: Writing tests after the fact to "cover" code. (Write tests alongside. If you can't test it during implementation, you don't understand it.)
- BAD: "Let's defer tests to a follow-up PR." (Tests are the cheapest thing to write with AI. Do it now.)
- BAD: "I'll just implement the happy path for now." (Edge cases cost seconds with AI. Handle them.)
