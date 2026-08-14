# Usage Guide

## Daily Workflow

### 1. Start a Session

Open Ghostty. Main pane:

```bash
cd ~/your-project
claude
> /kickoff
```

This reads your last `session-handoff.md` + `roadmap.md` + git log, then builds a numbered task queue.

### 2. Pick Tasks & Create Worktrees

From the task queue, pick 1-2 independent tasks to work in parallel:

```bash
# Exit claude (Ctrl+C) or open new panes

# Create worktrees for parallel work
git worktree add ../project-auth -b feat/auth
git worktree add ../project-api -b fix/api
```

Split Ghostty panes (Cmd+D horizontal, Cmd+Shift+D vertical):

```
┌─────────────────┬──────────────┐
│ Main pane       │ Worktree A   │
│ (orchestrator)  │ claude       │
│ stays on main   │              │
│ no code changes ├──────────────┤
│                 │ Worktree B   │
│                 │ claude       │
└─────────────────┴──────────────┘
```

### 3. Build Features

In each worktree pane:

```bash
cd ../project-auth
claude
> /feature add Google OAuth authentication
```

The feature flow runs: **plan → implement → /verify (loop) → review → commit**

### 4. Create PRs

When a feature is done and committed:

```bash
# Still in the worktree pane
git push origin feat/auth
gh pr create --title "feat(auth): add Google OAuth" --body "description"
```

Or just tell Claude: "create a PR for this"

### 5. Review & Merge

- Open GitHub in browser
- Review the PR diff (file changes, additions/deletions)
- Merge with the green button
- Repeat for each worktree

### 6. Clean Up & Handoff

Back in the main pane:

```bash
# Pull merged changes
git pull origin main

# Remove worktrees
git worktree remove ../project-auth
git worktree remove ../project-api

# Hand off to next session
claude
> /handoff
```

This writes `session-handoff.md` for tomorrow's `/kickoff` to read.

---

## Sequential Workflow (Single Feature)

For quick tasks or when starting out, skip worktrees:

```bash
cd ~/your-project
claude
> /kickoff            # plan the session
> /feature [desc]     # build it (includes verify + review)
> /handoff            # wrap up
```

---

## Skill Reference

| Command | When to Use |
|---------|-------------|
| `/kickoff` | Start of every session |
| `/feature [desc]` | Build one feature end-to-end |
| `/verify` | Test code works (auto-runs in /feature, can run standalone) |
| `/techdebt` | End of session or when codebase feels messy |
| `/handoff` | End of every session |
| `/research [topic]` | Before building something unfamiliar |
| `/design [project]` | New project — establish design system |
| `/railway [desc]` | Deploy to Railway |
| `/cloudflare [desc]` | Set up Cloudflare Workers/D1/KV |
| `/oauth [provider]` | Set up auth (Google, Apple, Supabase) |

Auto-loaded skills (no command needed):
- **commit-conventions** — enforced when committing
- **code-standards** — enforced when writing code
- **nextjs-patterns** — loaded in Next.js projects
- **python-fastapi-patterns** — loaded in FastAPI projects

---

## Agent Teams (Experimental)

For large tasks that decompose into parallel work:

```bash
# Add to ~/.zshrc
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1

# Then in Claude, just describe the parallel work:
> "Review this codebase — one agent checks security, another checks performance"
> "Build the chat feature — one agent does the API, another does the iOS view"
```

Use sparingly — burns tokens fast. Best for audits and complex multi-layer features.

---

## Tips

- **Don't change code in the main pane** — it's your orchestrator
- **One feature = one branch = one PR** — always
- **If a task grows too big**, tell Claude to split it
- **If scope creeps** ("while we're here..."), note it for `/techdebt`, stay focused
- **Review PRs on GitHub** — that's your quality gate before merge
