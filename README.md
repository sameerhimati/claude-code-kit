# claude-code-kit

Shared Claude Code skills, agents, and rules by [@sameerhimati](https://github.com/sameerhimati). Reusable across all projects. This repo mirrors the live `~/.claude` setup; anything retired lives in [`legacy/`](legacy/).

## Install Globally

```bash
git clone git@github.com:sameerhimati/claude-code-kit.git ~/claude-code-kit

mkdir -p ~/.claude/skills ~/.claude/agents ~/.claude/rules

for skill in ~/claude-code-kit/skills/*/; do
  ln -sf "$skill" ~/.claude/skills/$(basename "$skill")
done

for agent in ~/claude-code-kit/agents/*.md; do
  ln -sf "$agent" ~/.claude/agents/$(basename "$agent")
done

for rule in ~/claude-code-kit/rules/*.md; do
  ln -sf "$rule" ~/.claude/rules/$(basename "$rule")
done
```

## Skills

| Command | Description |
|---------|-------------|
| `/session-kickoff` | start a session: read last handoff + git state, build a task queue |
| `/session-handoff` | end a session: persist context for the next one |
| `/feature-flow` | build a feature: plan → implement → verify → review → commit |
| `/autoloop` | make the goal verifiable (test/lint/build), then loop until green |
| `/verify` | lint, type-check, tests, build, server smoke test with a fix loop |
| `/investigate` | root-cause debugging in five phases; no fixes without a root cause |
| `/qa` | systematically QA a web app, then fix what's found, commit by commit |
| `/design-review` | designer's-eye QA on a live UI; fixes with before/after screenshots |
| `/ux-audit` | walk the live app as a persona; catalog friction, then fix it |
| `/glow-up` | de-slop a UI in four waves until a scored rubric passes |
| `/prototype` | turn an idea into a working clickable prototype |
| `/deslop` | strip the tells that mark prose as LLM-written; opt-in plain-English mode |
| `/research` | parallel subagent research across web, codebase, community |
| `/think` | open-ended strategic thinking partner |
| `/office-hours` | YC-style forcing questions, or builder-mode brainstorming |
| `/learn` | interactive learning with six modes, backed by a knowledge vault |
| `/compile` | distill reading into post drafts and atomic notes |
| `/paper` | triage an arXiv paper from its LaTeX source |
| `/lint` | knowledge-vault health check and weekly review |
| `/plan-ceo-review` | founder-mode plan review: rethink scope and ambition |
| `/plan-eng-review` | eng-manager plan review: architecture, edge cases, tests |
| `/plan-design-review` | design critique of a plan, scored per dimension |
| `/plan-devex-review` | developer-experience review for APIs, CLIs, SDKs, docs |
| `/oss` | make-a-repo-public audit: secrets, docs, license, stranger test |
| `/deploy` | deploy a project to Railway or Cloudflare Workers |
| `/config-audit` | maintain `~/.claude` itself: drift, bloat, dead skills |
| (auto) | **claude-api** — Claude API reference: models, pricing, caching, tool use |
| (auto) | **use-railway** — operate Railway infra: projects, services, domains, debugging |

Conventions for writing skills: [`skills/_conventions.md`](skills/_conventions.md) · lint them with [`skills/_skill-lint.md`](skills/_skill-lint.md).

## Agents

| Agent | Description |
|-------|-------------|
| explorer | read-only codebase mapper; condensed findings with file:line refs |
| planner | codebase exploration → implementation plan |
| researcher | multi-source web research; cited, decision-ready synthesis |
| reviewer | code review for bugs, style, security, performance |
| security-scanner | quick vulnerability scanning |
| security-auditor | deep security audit targeting AI-generated code weaknesses |
| doc-updater | update docs after feature completion |
| skill-author | author and edit skills, rules, and agent definitions |
| config-auditor | read-only `~/.claude` audit; backs `/config-audit` |

## Rules

Always-on standards, loaded automatically rather than invoked:

| Rule | Scope |
|------|-------|
| [`code-standards`](rules/code-standards.md) | every repo — simplicity, minimal diffs, testing, observability |
| [`commit-conventions`](rules/commit-conventions.md) | every repo — messages explain why; one logical change per commit |
| [`nextjs`](rules/nextjs.md) | Next.js repos only |
| [`python-fastapi`](rules/python-fastapi.md) | FastAPI repos only |
