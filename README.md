# claude-code-kit

Shared Claude Code skills and agents by [@sameerhimati](https://github.com/sameerhimati). Reusable across all projects.

## Install Globally

```bash
git clone git@github.com:sameerhimati/claude-code-kit.git ~/claude-code-kit

mkdir -p ~/.claude/skills ~/.claude/agents

for skill in ~/claude-code-kit/skills/*/; do
  ln -sf "$skill" ~/.claude/skills/$(basename "$skill")
done

for agent in ~/claude-code-kit/agents/*.md; do
  ln -sf "$agent" ~/.claude/agents/$(basename "$agent")
done
```

## Skills

| Command | Description |
|---------|-------------|
| (auto) | **commit-conventions** — commit messages, branch naming, PR format |
| (auto) | **code-standards** — naming, error handling, file org, testing |
| `/kickoff` | **session-kickoff** — decompose work into a numbered task queue |
| `/feature [desc]` | **feature-flow** — plan → build → test → review → commit |
| `/techdebt` | **techdebt-scanner** — find duplication, dead code, TODOs |
| `/handoff` | **session-handoff** — generate handoff prompt for next session |
| `/verify` | **verify** — run, test, spin up server, loop until green |
| `/research [topic]` | **research** — parallel subagent research |
| `/deslop [file]` | **deslop** — strip the tells that mark prose as LLM-written, without rewriting the author's voice; opt-in plain-English mode |
| `/design [project]` | **design-philosophy** — interactive Q&A → generates design.md |
| `/railway [desc]` | **railway-deploy** — deploy to Railway without leaving terminal |
| `/cloudflare [desc]` | **cloudflare-setup** — Workers, D1, KV setup and deploy |
| `/oauth [provider]` | **oauth-setup** — step-by-step OAuth provider setup with code gen |
| (auto) | **nextjs-patterns** — Next.js App Router conventions |
| (auto) | **python-fastapi-patterns** — FastAPI conventions |

## Agents

| Agent | Model | Description |
|-------|-------|-------------|
| reviewer | Sonnet | Code review for bugs, style, security, performance |
| planner | Opus | Codebase exploration → implementation plan |
| doc-updater | Haiku | Update docs after feature completion |
| security-scanner | Sonnet | Quick vulnerability scanning |
| security-auditor | Opus | Deep security audit targeting AI-generated code weaknesses |
