---
name: lint
description: >
  Vault health check, weekly review coaching, and todo generation for the knowledge vault.
  Conversational — Claude guides Sameer through, asks questions, generates artifacts.
  Invoke manually with /lint. Use when asked to "lint the vault", "weekly review",
  "vault health check", "generate todos", or on Sundays.
disable-model-invocation: false
allowed-tools: Bash, Read, Write, Edit, Glob, Grep, AskUserQuestion, ExitPlanMode
---

# Vault Lint: $ARGUMENTS

You are running a vault health check and weekly review for Sameer's knowledge vault at `~/Desktop/knowledge/`.

This is a **conversation**, not a report. Ask questions. Wait for answers. Push back when something doesn't add up. You're a coach, not a CI pipeline.

**Time budget:** 30 minutes max. If the vault is healthy, say so and move on.

---

## Step 0: Read Vault State

Before anything, gather context silently:

1. Read `CLAUDE.md` (vault conventions)
2. Read `index.md` (routing doc)
3. Read `projects/_Dashboard.md` (project statuses)
4. Read `log.md` (recent operations)
5. Find the most recent weekly review in `daily/`
6. Find the most recent `W##-todo.md` in `daily/`
7. Glob all `.md` files to understand vault size

Do NOT dump all of this to Sameer. Internalize it. You'll reference it throughout.

---

## Step 1: Vault Health Check

Scan for issues. Present findings conversationally — not as a wall of text.

### What to check:

**Orphan notes:** Find notes with zero incoming wikilinks. Exclude: index.md, _Dashboard.md, _Weekly-Review.md, templates, daily notes, log.md.
- Method: Grep all `[[wikilinks]]` across the vault, collect targets, diff against all filenames.

**Broken wikilinks:** Find `[[link-targets]]` that don't resolve to any file.
- Method: Extract all wikilink targets, check if a matching `.md` file exists. Red links in Obsidian terms.
- **Heavy false-positive caveat (learned 2026-05-31):** a naive basename scan over-reports massively. Before flagging, discount: (a) path-style links like `[[clinic/about]]`, `[[projects/_Dashboard]]` (resolve fine in Obsidian); (b) anything under `writing/` (it's a symlink — `find .` won't traverse it); (c) intentional learning-map curriculum stubs (`[[bayes-theorem]]`, etc. — planned-not-yet-written, by design); (d) memory-slug refs (`[[project_*]]`, `[[feedback_*]]` live in `~/.claude`, not the vault); (e) example text in docs. Report only the *real* residue, not the raw count.

**Reading-queue signal pass:** Apply the two-lane filter at the top of `inbox/reading-queue.md` (SIGNAL + EXPLORE; kill overlap/rot/superseded). See [[feedback_reading_filter]]. Also: read items that are durable graduate to `research/_canon.md` (the queue stays *pure unread*); disposable read items just fall off. New bookmarks clear the same bar before entering. Don't gut breadth — EXPLORE is a protected lane.

**Stale pages:** Notes in `projects/` folders of Active projects not modified in 14+ days.
- Method: Check file modification times against Dashboard status.

**Inbox aging:** Files in `inbox/` and `inbox/raw/` older than 7 days that haven't been processed.
- Method: Check dates, flag anything sitting too long.

### How to present:

- Lead with the headline: "Vault looks healthy" or "Found a few things"
- Group by severity: broken links first, then orphans, then staleness
- For each finding, propose a fix: "This orphan note could link from [[X]]" or "This broken link — did you mean [[Y]]?"
- Ask: "Want me to fix these, or should we move on?"

---

## Step 2: Weekly Review Coaching

Walk through the weekly review template conversationally. Reference `projects/_Weekly-Review.md` for structure.

### The conversation:

1. **What shipped this week?**
   - Don't accept "nothing" without pushback. Check git history if needed.
   - Celebrate what got done, even small things.

2. **What did you learn?**
   - From reading, building, conversations, mistakes.
   - If something is worth capturing, offer to create a vault note.

3. **Active project check-in** (for each Active project on the Dashboard):
   - "What happened on [project] this week?"
   - "What's the next action?"
   - "Are you blocked on anything?"

4. **Maintenance check:**
   - Deployed apps running?
   - Invoicing due?
   - Anything need attention?

5. **How are you feeling?**
   - Energy, motivation, clarity.
   - Be honest. Push if the answer is vague.

6. **One thing you're avoiding.**
   - Name it. Don't let him skip this.

7. **Top 3 priorities for next week.**
   - These become the spine of the W## todo.

### Output:
Write the completed review to `daily/YYYY-MM-DD-weekly-review.md` with proper frontmatter:
```yaml
---
date: YYYY-MM-DD
week_number: W##
---
```

---

## Step 3: Weekly Planning — Two Passes, Plan-First

Build the week **top-down and interactively**: surface everything, agree the backlog, *then* allocate to days. Do NOT one-shot a todo file — propose, ask, adjust, confirm. The *what* gets locked before the *when*.

### Step 3a: Surface off-vault commitments FIRST

Before planning anything, ask explicitly — don't assume the vault is the whole picture:

> "Before I build next week — what else is on your plate? Meetings, deadlines, commitments, appointments, anyone you owe something, anything time-boxed that isn't already in the vault?"

The review (Step 2) covers what's *in* the vault; this catches what isn't. Probe for: scheduled calls/interviews, travel, social, recurring beats (Sutro Mondays, etc.), external deadlines, errands that must land on a specific day. Pull every answer into the working backlog.

### Step 3b: Pass 1 — Week level (the backlog)

Assemble the **complete** list of the week's tasks, goals, and commitments. Sources: the review's top priorities, **carried-over items** from the most recent `W##-todo.md`, the writing roadmap's "next up," and the off-vault items from 3a.

- Present it as one backlog, grouped into the week's **natural themes** (keystone / build / apply / read / people / housekeeping — themed to the actual week, not a fixed template).
- Name the **keystone** — the one non-negotiable thread the week orbits.
- Confirm it: "This is everything I have for the week — what's missing, what's not real, what's actually parking-lot?" Adjust until he agrees the backlog is complete and honest.

Do NOT allocate to days yet.

### Step 3c: Pass 2 — Day level (allocate, in plan mode)

Only once the backlog is confirmed, allocate it across the week. **Present this in plan mode and get sign-off (ExitPlanMode) before writing anything.**

For each day, spell out concretely **what that day looks like** — this per-day framing is the point:

> Monday looks like: Sutro tonight + the daily keystone hour.
> Tuesday looks like: the one scoped build box, then done.
> …

- Respect fixed commitments from 3a (a Thursday call lands on Thursday).
- Protect the keystone — it gets its block *every* day, not whatever's left over.
- Keep each day realistic: one to a few concrete things, not a wish-list.
- If he reshuffles days at sign-off, fix it here — before it's written.

### Step 3d: Write the plan

After sign-off, write both:

1. **`daily/2026-W##-todo.md`** — the backlog in his format: a leading **mode** line (the week's framing/keystone in one breath), themed sections, and a **Parking lot — capture only, do NOT start** block at the bottom. Match the most recent `W##-todo.md`'s voice and shape.
2. **Seed each daily note** (`daily/YYYY-MM-DD.md`) with that day's allocation — so the week opens with no blank pages.

   **→ The canonical shape now lives in `systems/daily-note-template.md`. Read it and follow it.** Two rules it enforces, both from corrections Sameer issued 2026-08-10:
   - **Three to five tickable items per day, never more.** Habits are one line of prose, not checkboxes. A ten-box day is a contract and gets ignored → `feedback_daily_schedule_design`.
   - **Resolve every session to a real link.** Compute the lift rotation for the whole week at lint time (oldest logged date first across `upper-a` · `lower-a` · `upper-b` · `lower-b`) and write the actual `[[upper-b]]` into that day. Never make him look up which one is next — a lookup at 6am is a reason to skip.
   - **Don't collect data that feeds a report instead of a decision.** No step counts, no habit boxes, no daily weigh-in row. Lift loads only → `feedback_unchecked_boxes_are_not_evidence`.

   <details><summary>Superseded structure (kept for history — do not use)</summary>

   ```
   # YYYY-MM-DD (Day)

   ## Habits
   - [ ] Weight (AM, fasted) → [[weight-log]]
   - [ ] 5am up
   - [ ] PPL lift · protein + IF window
   - [ ] 15K steps
   - [ ] Job rep — 1 human OR 1 leetcode
   - [ ] 5-min breathing + AM sunlight

   ## Blocks
   - [ ] <full sleep-anchored time-block per [[daily-routine]]: wake · lift · keystone · meals · cardio · 9:15 lights-out>
   ...

   ---

   ## Today
   > Focus: <one-line focus>
   - [ ] <concrete deliverable / task for the day>   ← the *what* (tickable checklist)
   - [ ] <commitment / call / errand>
   ...
   ```

   Blocks = the *when*, Today = the *what*. Derive the Today checklist from the day-level pass (3c) + that day's themes. A few concrete, completable items — keystone first.

   </details>

### Rules:
- Each item concrete and completable in one session.
- Don't overload — a few items per day, keystone first.
- Carried-over items get pulled forward, not silently dropped.

---

## Step 4: Dashboard Sync

Compare what was said in the review to the Dashboard statuses.

- Flag discrepancies: "You said you haven't touched [project] in 3 weeks but it's still Active. Move to Paused?"
- Check the 4-project Active limit: if more than 4 are Active, something needs to move
- **Never auto-update the Dashboard.** Present changes, get confirmation, then edit.

---

## Step 5: Writing Roadmap Check

Review the writing roadmap at `systems/operating/writing-roadmap.md`:

1. **Status update:** Walk through each item on the roadmap. What shipped this week? What moved forward? Update statuses (e.g., mark as published, update target dates).
2. **Freshness check:** Are there new learnings, projects, or reads from this week that should become a new entry on the roadmap?
3. **Sequencing:** Does the order still make sense? Anything more urgent now? Anything that should be parked or killed?
4. **Next up:** Confirm which piece Sameer is writing next week. Add it to the W## todo's Write section.

Also scan `research/` and `ideas/` for substantial notes (>20 lines, well-linked) that could become new roadmap entries.

This step is NOT optional — the writing compounds like the vault does.

---

## Step 5.5: Fitness & Progress Report

Read `systems/workouts/weight-log.md` (daily weigh-ins) and the six PPL workout Log tables (`systems/workouts/monday-push.md` … `saturday-legs.md`). Produce a short weekly progress report — a coach's check-in, not a spreadsheet:

1. **Weight trend** — this week's avg vs last week's rollup, net Δ, and the 7-day direction (the trend beats any single day). Flag if off-track toward the 220→200 goal.
2. **Training adherence** — how many of the 6 lifts + 3 cardio sessions actually happened (from logged loads + daily checkboxes). Name the misses honestly.
3. **Chart** — generate a simple weight-over-time line chart: write a small matplotlib script to a scratch file, run it with Bash, save the PNG to `systems/workouts/charts/weight-YYYY-Www.png`, and embed it in the report (`![[weight-YYYY-Www.png]]`). If plotting isn't available, fall back to an inline table / sparkline.
4. **One improvement** — the single highest-leverage fix for next week (sleep, protein, a lagging lift, adherence).

Append a one-row rollup to the `## Weekly rollups` section of `weight-log.md`: week · avg weight · Δ vs last week · adherence · the one improvement.

---

## Step 6: Log Update

Append a lint entry to `log.md`:

```
## [YYYY-MM-DD] lint | Weekly vault health check
Health: [X orphans, Y broken links, Z stale pages]. Review: W## completed. Todo: W## generated. Dashboard: [changes or "no changes"]. Report: progress-W## generated.
```

---

## Step 7: Weekly Progress Report (glanceable HTML artifact)

End the lint by handing Sameer something he can **see** — a self-contained HTML report he glances at, not a spreadsheet. Gym is what he cares about most; weight is secondary. Keep it simple: a few clean charts + a short summary. Do NOT invent numbers — chart only what's logged.

**Reuse, don't recompute.** The weight trend, training adherence, and one-improvement line were already computed in Step 5.5. Carry those numbers straight in. This step adds the gym load progression and the steps/job-rep signals, and renders the whole thing visually. It's the glance layer; Step 5.5's rollup row in `weight-log.md` stays the durable text record.

### Build it

1. **Read the template:** `~/.claude/skills/lint/report-template.html`. It's a finished, self-contained page (inline CSS/JS, theme-aware light/dark, colorblind-safe palette, wide charts scroll on mobile, hand-rolled SVG — no external deps). You only edit the JSON inside `<script id="report-data">`. Leave all other markup and the render script untouched.

2. **Fill the JSON from real logs** (shape and field names are documented by the default JSON already in the template):
   - `weight.points` — this week's rows from `weight-log.md` (`{d, w}`). `goalLo: 200`.
   - `lifts` — the key compounds only (Bench, OHP, Barbell Row, Incline DB, Chin/Pull-ups, Hack/Leg Press, Trap Bar DL — pick what was trained). For each, `points` = the **top working-set load** per session across the recent logs (loads accumulate over weeks; one session is fine, the sparkline fills in). `reps` = a short string like `"5×4"`. Set `pr:true` when this week beat the last logged load for that lift.
   - `adherence.lifts` — one entry per PPL day (`{day, type, done}`), `done` from the daily-note `PPL` habit checkbox + a logged row. `cardioDone`/`cardioTarget` (target 3).
   - `steps` and `jobs` — see the gap note below.
   - `summary` (one honest paragraph) and `improve` (the Step 5.5 one-liner).

3. **Write** the filled file to `systems/reports/progress-YYYY-Www.html` (create `systems/reports/` if absent), and **render it as an Artifact** so Sameer sees the charts inline.

### Data gap — steps & jobs are NOT logged numerically (flag this)

Only binary signals exist today: the `15K steps` and `Job rep` habit checkboxes in each daily note. So:
- **Steps:** set `steps.numeric:false` and fill `steps.days` from the checkboxes — the report shows 15K hit/miss and prints the fix.
- **Jobs:** set `jobs.numeric:false` and fill `jobs.days` from the checkboxes; `funnel:null` — the report shows rep hit/miss and prints the fix.

The template already prints the proposed convention: **one optional line per daily note** —
`metrics: steps=16.2 apps=9 humans=1 interviews=0 artifacts=0` (steps in thousands, fill any you have). Once these lines accumulate, set `numeric:true` and pass `steps.counts` / `jobs.funnel`, and the placeholders become real charts. **Mention this to Sameer once** — don't nag, and don't build any heavier tracking. It's opt-in; the report is honest and useful without it.

---

## Rules

- **This is a conversation.** Ask questions. Wait for answers. Don't monologue.
- **Never auto-fix without asking.** Present findings, propose changes, get confirmation.
- **If Sameer says "just do it all"** — still summarize what you're about to do before doing it.
- **Keep it under 30 minutes.** If the vault is healthy and the review is quick, that's great.
- **Push back.** If a project has been "Active" for 3 weeks with no progress, say so. If the inbox is overflowing, say so. Coach, don't placate.
- **Always end by asking:** "Anything else bugging you about the vault?"
