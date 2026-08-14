---
name: config-audit
description: Periodic maintenance of the ~/.claude configuration — deletion-tests CLAUDE.md, finds stale facts and dead/broken skills, measures real skill usage, checks for conflicts, and proposes fixes. Invoke with /config-audit. Run every few weeks to keep this folder from drifting.
disable-model-invocation: false
allowed-tools: Read, Glob, Grep, Bash, Edit, Write, Agent
---

# Config Audit

Keep `~/.claude` lean, accurate, and unbroken. This is the periodic-maintenance loop for the master config.

## Step 1: Run the audit
Spawn the `config-auditor` agent (read-only; returns a findings table). If unavailable, run the checks inline:
1. **CLAUDE.md** — `wc -l ~/.claude/CLAUDE.md` (target ~200); deletion-test each line; flag emphasis overuse.
2. **Stale facts** — verify paths in CLAUDE.md exist; reconcile the Projects list against `ls ~/Code`.
3. **Usage** — count `/skill` invocations from the `display` fields of `~/.claude/history.jsonl`, e.g. `rg -oP '(?<=\s|^)/[a-z][a-z-]+\b' ~/.claude/history.jsonl | sort | uniq -c | sort -rn`. Never read history.jsonl whole — counts only. Flag zero-use skills.
4. **Integrity** — `find ~/.claude/skills ~/.claude/agents -type l ! -exec test -e {} \; -print` for broken symlinks; cross-check advertised skills vs SKILL.md files on disk.
5. **Conflicts** — scan CLAUDE.md, rules/, and skills for contradictory instructions.
6. **Skill lint** — run the 7-point lint in `~/.claude/skills/_skill-lint.md` across `~/.claude/skills/*/SKILL.md`; report per-skill pass/fail in the findings table. This subsumes the usage-count (lint test 6), integrity (lint test 3), and conflict (lint test 2) checks above.

## Step 2: Present findings
Show a findings table: issue → severity → recommended action. Separate "stale fact / breakage" (fix now) from "judgment call" (skill cut/merge — ask first).

## Step 3: Apply (with consent)
Fix breakage and stale facts directly, but back up CLAUDE.md first: `cp ~/.claude/CLAUDE.md ~/.claude/backups/CLAUDE.md.bak-$(date +%Y%m%d)`. For skill cuts/merges or anything subjective, ask before changing — per the routing rule, don't auto-restructure.

## Iron law
Back up CLAUDE.md before editing it. Never delete a skill without confirming it's truly unused (zero history hits) AND not a deliberately-dormant domain template.

End with a completion status (DONE / DONE_WITH_CONCERNS / BLOCKED / NEEDS_CONTEXT).
