---
name: glow-up
description: >
  De-slop a project's UI/UX. Orchestrates a four-wave transformation
  (identity → consistency → polish → microcopy) and loops design-review +
  ux-audit until a scored rubric crosses a threshold or a hard cap fires.
  Use when an app "looks AI-generated" and needs to look made by humans.
  Invoke with /glow-up. Pair with /loop for unattended runs.
allowed-tools: Bash, Read, Write, Edit, Glob, Grep, Agent, AskUserQuestion, WebFetch, WebSearch
---

Reference: see ~/.claude/skills/_conventions.md for shared patterns.

# /glow-up: AI-slop → Well-Made

You are the orchestrator that transforms a project's visual identity from
"obviously AI-generated" to "obviously made by someone who cares." You do
not invent new design checks — you compose `/design-review` and `/ux-audit`
into a scored loop with a clear stop condition.

The pattern this skill encodes is the **clinic playbook** (validated on a
real project, March–April 2026):

1. **Identity** — DESIGN.md from competitive research + custom font + named palette
2. **Consistency** — sweep across files for tracking, focus rings, tabular nums
3. **Polish** — redesign one hero surface to a higher bar
4. **Microcopy** — kill "N/A", make button labels specific, design empty states

Each wave moves a measurable rubric. Loop fires until the rubric crosses 8/10
average or the iteration cap (default 5) is reached.

## IRON LAWS

- **No loop without a rubric.** "Until happy" without a scorecard is token-burn. Score before, score after, every iteration.
- **One wave at a time.** Don't mix identity changes with polish changes — you can't attribute what helped.
- **Stop on diminishing returns.** Two consecutive iterations with <0.5 score gain → stop.
- **Hard cap: 5 iterations.** No exceptions.
- **Don't re-implement design-review or ux-audit.** Call them. If you find yourself writing visual checklists, you're scope-creeping.
- **Clean working tree before each wave.** Each wave is its own commit batch.

---

## Step 0: Parse Arguments

- `/glow-up` — auto-detect URL, full four-wave run, cap 5
- `/glow-up <url>` — run on a specific URL
- `/glow-up --wave identity` — run only the identity wave
- `/glow-up --wave consistency` — run only the consistency sweep
- `/glow-up --wave polish` — run only hero-surface polish
- `/glow-up --wave microcopy` — run only microcopy wave
- `/glow-up --score` — score only, don't fix
- `/glow-up --threshold 9` — raise stop threshold (default 8/10)
- `/glow-up --cap 3` — lower iteration cap (default 5)

For unattended runs, the user can wrap this skill in `/loop`:
- `/loop /glow-up` — model self-paces re-scoring iterations
- `/loop 4h /glow-up --wave consistency` — sweep every 4h on a long-running project

---

## Step 1: Project Read + Working-Tree Check

```bash
# Working tree must be clean — each wave commits its own changes
git status --porcelain
```

If dirty, use AskUserQuestion to offer commit/stash/abort (same pattern as /design-review).

```bash
# Detect project type (web app required)
test -f package.json && grep -q '"next"\|"react"\|"vue"\|"svelte"' package.json && echo "WEB_APP=true"

# Find the design system doc (or note its absence)
ls DESIGN.md design-system.md docs/design.md 2>/dev/null

# Detect dev server / target URL
git diff main...HEAD --name-only 2>/dev/null | head -10  # for diff-aware mode
```

If not a web app, STOP — `/glow-up` only works on rendered web UIs.

If no URL given, ask: local dev server? deployed staging? prod?

---

## Step 2: Baseline Score

Score the current state of the app on the 5-dimension rubric. **Do this BEFORE any changes** — this is the number we're trying to move.

### The Rubric (0–10 per dimension, 8.0 average = stop)

| Dimension | What it measures | How to score |
|-----------|------------------|--------------|
| **Identity** | Has the project broken from defaults? | DESIGN.md exists (+3), custom font not Geist/Inter/Roboto (+3), named palette (not raw shadcn slate/blue) (+4) |
| **Consistency** | Do patterns hold across files? | Heading scale uniform (+2), tabular-nums on financials (+2), focus-visible everywhere (+2), no `transition: all` (+2), spacing on a scale (+2) |
| **AI slop absence** | Did anti-patterns get removed? | No purple gradients (+2), no 3-column icon-grid (+2), no centered-everything (+2), no uniform bubbly radius (+2), no decorative blobs (+2) |
| **Hero polish** | Is at least one surface intentionally designed? | Identify the most-used surface. Score 0–10 against `/design-review` Phase 6 rubric on that page only |
| **Microcopy** | Are words specific and warm? | No "N/A" visible (+3), button labels are verbs+nouns (+3), empty states have action+illustration (+2), error messages have fix (+2) |

### How to score

Spawn a `general-purpose` agent (in parallel where possible) per dimension:

```
Agent (Identity): Read DESIGN.md if present. Read app/layout.tsx, globals.css, tailwind.config. Report: does this project have visual identity beyond shadcn defaults? Score 0-10 with specific evidence (font name, primary color hex, palette names).

Agent (Consistency): Grep across src/ for: heading classes, transition-all, font-mono usage, focus-visible. Report: how consistent are these patterns? Score 0-10 with file:line examples of inconsistency.

Agent (AI slop): Run /design-review --quick (or call browse for screenshots if available). Report only the AI Slop Score and the specific anti-patterns found. Score 0-10 (10 = zero anti-patterns).

Agent (Hero polish): Identify the highest-traffic surface from routes and CLAUDE.md. Read its component file. Score 0-10 against /design-review's category checklist for that page only.

Agent (Microcopy): Grep for "N/A", "Submit", "Continue", "No items", "Loading...". Report counts and locations. Score 0-10 (10 = zero generic copy).
```

Aggregate and write `BASELINE` to `~/.claude/glow-up-runs/<project>-<date>/baseline.json`:

```json
{
  "date": "YYYY-MM-DD",
  "project": "<name>",
  "url": "<target>",
  "scores": { "identity": 4, "consistency": 5, "ai_slop": 6, "hero_polish": 4, "microcopy": 5 },
  "average": 4.8,
  "evidence": { ...per-dimension specifics... }
}
```

Show the user the scorecard before starting any waves. They may want to redirect — e.g., a project may already have strong identity but weak consistency, in which case you skip Wave 1.

---

## Step 3: Wave Selection

Pick the wave that targets the lowest-scoring dimension. Don't run waves in fixed order — run them in score-ascending order.

| Lowest dimension | Wave to run |
|------------------|-------------|
| Identity | Wave 1: Identity |
| Consistency | Wave 2: Consistency |
| AI slop | Wave 3: AI-slop sweep (calls /design-review) |
| Hero polish | Wave 4: Hero polish |
| Microcopy | Wave 5: Microcopy |

Show the user: "Lowest score is `<dimension>` at `<N>`/10. Running Wave `<X>`. Estimated `<Y>` files touched. OK?"

---

## Wave 1: Identity (target: identity score → 9+)

**Goal:** Project breaks visibly from shadcn-default look.

### 1a. Competitive research

If DESIGN.md doesn't exist, spawn a research agent:

```
Agent (research): The user is building <project description from CLAUDE.md>. Research 3-5 closest competitors. For each, capture: primary font, primary brand color (hex), accent color, overall mood (clinical/playful/bold/calm). Web search and visit their marketing sites. Output a short comparison table.
```

### 1b. Pick the inversion

Show the user the competitor table and propose:
- One **custom font** that's neither Geist nor Inter (Plus Jakarta Sans, Söhne, Söhne Mono, IBM Plex, Manrope, Geist Mono are fine — but it must be a deliberate pick, not the default).
- One **primary color** that has a name and is not raw `blue-600`.
- One **accent** that pairs with the primary (warm with cool, or vice versa).

Use AskUserQuestion. Recommend based on competitor research and project mood from CLAUDE.md.

### 1c. Create DESIGN.md

Write a structured DESIGN.md to repo root. Mirror the structure used by the clinic project:

```
# Design System — <project name>
## Product Context
## Aesthetic Direction
## Typography (with named scale)
## Color (with named palette and usage rules)
## Spacing (with named scale)
## Layout
## Motion (anti-patterns explicitly listed)
## Anti-patterns (the AI-slop blacklist)
```

### 1d. Apply the swap

Edit `globals.css`, `app/layout.tsx`, `tailwind.config.*`:
- Swap font imports.
- Update CSS variables to the new palette.
- Run `grep -rE "(blue|gray|slate)-(50|100|...)\b" src/` to find hardcoded color classes — migrate to the new palette.

### 1e. Commit + re-score Identity dimension only

```bash
git add DESIGN.md clinic-app/src/app/globals.css clinic-app/src/app/layout.tsx ...
git commit -m "feat(design): identity — <font> + <palette> from DESIGN.md"
```

Re-run the Identity scorer. If still <9, ask user; otherwise advance.

---

## Wave 2: Consistency (target: consistency score → 9+)

**Goal:** The patterns from DESIGN.md hold across every file.

### 2a. Define the sweep checklist

From DESIGN.md, extract the rules that should be uniform:
- Heading sizes/weights
- Tabular numerals on number columns
- Focus-visible rings on custom interactive elements
- Specific transition properties (no `transition-all`)
- Spacing on the named scale
- Border-radius hierarchy

### 2b. Run the sweep as a single coordinated edit pass

For each rule, grep across `src/` and fix every instance. Bundle into ONE commit per rule, not one commit per file.

```bash
git commit -m "style(design): consistency sweep — <rule> across N files"
```

### 2c. Re-score

Re-run the Consistency scorer. If still <9, identify which rule failed and re-sweep that one.

---

## Wave 3: AI-slop sweep

**Goal:** Drop AI-slop score to 9+ (i.e., zero or one anti-pattern remaining).

This wave is mostly a wrapper around `/design-review`:

```bash
# /design-review already has the AI-slop blacklist and the fix loop.
# Just invoke it scoped to the surfaces we care about.
```

Run `/design-review --quick` (or `--deep` if score is very low). Let it apply
its own fix loop. When it returns, capture its AI-slop grade and pull the
delta into our score.

If `/design-review` reverts or runs out of self-regulation budget, surface
its STATUS and stop the wave.

---

## Wave 4: Hero polish

**Goal:** One surface scores A on `/design-review` Phase 6, so the rest of the app has a target.

### 4a. Pick the hero surface

Highest-traffic page (from analytics, CLAUDE.md, or sidebar nav order). Default candidates: dashboard, primary list view, detail view.

### 4b. Redesign

This is the only wave that may touch component structure, not just CSS.

- Read the current hero component end-to-end.
- Sketch (in markdown comments to the user) the intended new layout.
- AskUserQuestion to confirm direction before writing code.
- Apply.
- Run the dev server, screenshot, show the user.
- Iterate up to 3 attempts. After 3, escalate.

```bash
git commit -m "feat(design): redesign <surface> — <one-line summary of approach>"
```

---

## Wave 5: Microcopy

**Goal:** Microcopy score → 9+.

### 5a. Sweep generic strings

```bash
grep -rn "N/A\|n/a\|No items\|Submit\|Continue\|Loading\.\.\.\." src/ --include="*.tsx" --include="*.ts"
```

For each match, propose a specific replacement:
- Submit → "Save changes" / "Add patient" / "Send invoice"
- Continue → "Review billing" / "Confirm appointment"
- No items → context-specific empty state
- N/A → either remove or replace with the actual missing-data UX (em-dash, "—", or "Not recorded")

Use AskUserQuestion to confirm replacements in batch.

### 5b. Empty-state pass

For each list/table component, ensure the empty state has:
- A specific message explaining what should appear here
- One primary action to populate
- Optional illustration/icon

Commit per component or per route group:
```bash
git commit -m "fix(copy): specific empty states + button labels in <area>"
```

---

## Step 4: Re-score and Decide

After every wave (not after every commit):

```
Iteration N:  identity 6→9  consistency 5→7  ai_slop 6→8  hero 4→4  microcopy 5→5
              average 5.2 → 6.6
```

### Continue conditions
- Average < threshold (default 8.0) AND
- Iteration count < cap (default 5) AND
- Last iteration moved average by ≥ 0.5

### Stop conditions
- Average ≥ threshold → DONE
- Iterations == cap → DONE_WITH_CONCERNS (list dimensions still below 8)
- Two consecutive iterations with <0.5 gain → DONE_WITH_CONCERNS (diminishing returns)
- Any wave returned BLOCKED → BLOCKED, escalate

If continuing: pick the new lowest-scoring dimension and run that wave.

Save iteration history to `~/.claude/glow-up-runs/<project>-<date>/history.json`.

---

## Step 5: Final Report

```
STATUS: DONE | DONE_WITH_CONCERNS | BLOCKED
PROJECT: <name>
URL: <target>
ITERATIONS: <N>
BASELINE → FINAL
  Identity:    <a> → <b>
  Consistency: <a> → <b>
  AI slop:     <a> → <b>
  Hero polish: <a> → <b>
  Microcopy:   <a> → <b>
  Average:     <a> → <b>

### What moved the score
- Wave 1 (Identity): +X.X — <one-line summary>
- Wave 2 (Consistency): +X.X — <one-line summary>
- ...

### What didn't budge
- <dimension> stuck at <score> — <hypothesis why>

### Recommended next
- Run /ux-audit on <persona> to find product-decision issues this skill couldn't see
- Run /design-review --regression in 2 weeks to detect drift
- <skill suggestions>
```

Append run metadata to `~/.claude/glow-up-runs/index.md` (one line per run) so
future invocations can detect prior runs and offer regression mode.

---

## Anti-patterns

- BAD: Running waves in fixed 1→2→3→4→5 order. (Score-ascending order is faster.)
- BAD: Re-implementing /design-review's checklist inside this skill. (Call it.)
- BAD: "I'll mix Identity and Microcopy in one wave to save time." (You will not be able to attribute what helped.)
- BAD: Running for 12 iterations because the threshold is 9. (Hard cap is 5. Lower the threshold or accept the result.)
- BAD: Spawning 5 fix agents in parallel. (Waves are sequential — each wave commits before the next runs, so the next wave sees a coherent tree.)
- BAD: Auto-running on `main`. (Always work on a feature branch — `feature/glow-up-<date>` if not already on one.)
- BAD: Skipping the baseline score because the user wants to "just start fixing." (Without a baseline, you can't show what moved.)

---

## Pairing with /loop

This skill is a one-shot orchestrator. It does NOT run as a cron job. If the
user wants periodic runs:

- `/loop /glow-up --score` — model self-paces a re-scoring loop, alerts when score drifts
- `/loop 24h /glow-up --score` — daily score drift detection
- `/loop /glow-up --wave consistency` — model self-paces consistency sweeps

The runtime ScheduleWakeup mechanism handles cadence. Don't re-implement scheduling here.
