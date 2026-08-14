---
name: paper
description: >
  Read a paper (from the arXiv LaTeX source, not the PDF) and produce either a triage verdict
  — skip/skim/read closely, grounded in which of Sameer's reading lanes it serves — or, for
  canon, a reading map. Writes to research/ai-ml/papers/<tag>.md and updates the index.
  Invoke with /paper <arxiv-url> or when he says "read this paper", "is this worth reading",
  "what's in this paper".
disable-model-invocation: false
allowed-tools: Bash, Read, Write, Edit, Glob, Grep, WebFetch, AskUserQuestion
---

# Paper: $ARGUMENTS

Read the paper properly — from the LaTeX source — and hand Sameer either a **map to read it with** or a **verdict that saves him from reading it at all**.

---

## The lanes — this is what the verdict is grounded in

Never ask "is this a good paper." Ask: **which lane does this serve, and what depth does that lane demand?** The lanes are what he's actually doing, not a taxonomy I invented:

| Lane | Where it lives | Depth it demands | Verdict tends to be |
|---|---|---|---|
| **IMPLEMENT** | `~/Code/ml-fluency` (5 tracks; `05-frontier-papers` has an ordered list), `~/Code/nanogpt-mlx` (rung-by-rung) | Derive it. Reimplement in ~30 lines. | **READ CLOSELY** + reimplement |
| **CANON** | `research/_canon.md` | Everything cites it; not having read it is a hole. He's reading it regardless. | **READ CLOSELY** — and switch to *map mode* (below) |
| **FLUENCY** | The frontier debate; interviews; his edge is **translation** | Mechanism + caveats. Enough to hold a defensible opinion. | **SKIM** — most papers land here |
| **FUEL** | `writing/`, the agent series, `research/reading-notes/` | The idea, not the method. | **SKIM** or **SKIP** |
| **EXPLORE** | `inbox/reading-queue.md` (breadth lane) | Curiosity. Breadth is fuel — don't punish a paper for being off-track. | usually **SKIM** |
| **AWARENESS** | nowhere — he just needs the fact | One sentence, then done. | **SKIP** |

**Read the lane docs before judging.** `ml-fluency/05-frontier-papers/README.md` is an *ordered list* — if the paper is on it, that's an IMPLEMENT-lane paper and the position in the list matters. `nanogpt-mlx/ROADMAP.md` says which rung he's on — a paper about rung 6 is not actionable when he's on rung 1.

**Sutro Group is the forcing function, not the finish line** — his own words in the ml-fluency README. It paces the IMPLEMENT lane. It is *not* the reason he reads, and the verdict must never be anchored to it alone.

His ML dial is **systems-design fluency**, not research or kernels. His edge is **translation** — explaining what a result means to people who build products. Most papers are FLUENCY-lane, and for those the honest verdict is SKIM.

---

## The other axis: category

The lane says *why he'd read it*. The **category** says *what it's about*. Both go on every paper — the index groups by category, the lane says what to do.

`architectures` · `optimization` · `agents` · `reasoning` · `systems` (distributed, quantization, serving, energy) · `evals` · `theory` (scaling laws, learning theory)

Extend the list if a paper genuinely doesn't fit — but reach for an existing one first. A category with one member forever isn't a category.

---

## Two modes

**TRIAGE mode (default)** — a frontier paper, a new result, something he might not read. The question is *should he*, and the output leads with a verdict.

**MAP mode (canon)** — a foundational paper everything references. **"Should I read this?" is the wrong question — he's reading it regardless.** A verdict here is worthless. What he needs is a *guide*: what to focus on, what's aged badly, what everyone misquotes, what the paper is actually arguing against. Switch to map mode when the paper is in `research/_canon.md`, is obviously foundational (Attention, Adam, ResNet, GPT-3), or he says "I'm reading this."

If it's genuinely ambiguous, ask him. Don't guess.

---

## Step 1: Get the source

Extract the arXiv ID: `arxiv.org/abs/2601.07372`, `/pdf/2601.07372v2`, or bare `2601.07372` → `2601.07372` (strip any `vN`).

Skip the download if `~/.cache/knowledge/arxiv/<id>/` exists and is non-empty.

```bash
ARXIV_ID=<id>
DEST=~/.cache/knowledge/arxiv/$ARXIV_ID
mkdir -p "$DEST"
curl -sL -A "paper-skill/1.0" "https://arxiv.org/src/$ARXIV_ID" -o "$DEST.tar.gz"
tar -xzf "$DEST.tar.gz" -C "$DEST" && rm "$DEST.tar.gz"
grep -rl --include='*.tex' '\\documentclass' "$DEST"   # the entrypoint
```

**Why source, not PDF:** equations survive, columns don't interleave, it's greppable, and `%`-commented lines show what the authors *cut*. **The entrypoint is often not `main.tex`** — it was `ms.tex` for *Attention Is All You Need* and `template.tex` for Muon. Grep, don't guess.

**No source available** (PDF-only, withdrawn, pre-2000s): `/src/` returns an error page or a bare PDF. Don't debug it. Say so in one line and read the PDF (`Read`) or the abstract page (`WebFetch`) instead.

**Not on arXiv at all?** Fine — a blog-post paper (Keller Jordan's Muon), a conference PDF, an ACM link. Read what you can get. Everything downstream is unchanged.

---

## Step 2: Read the paper

Start at the entrypoint, follow `\input`/`\include` through the body. Read the **actual argument** — setup, method, experiments, ablations. Skip the `.bib`, figure sources, and appendix unless the claim depends on them.

**Read the ablations and the limitations section with particular care.** That's where Step 4 comes from, and it's what a skim misses. In the Muon paper the buried negative result (Muon-SFT loses to Adam-SFT on a public checkpoint) was more important than the headline.

---

## Step 3: Gather his context — before you write a word of connection

You cannot say "this touches your work" until you've looked:

- **Lane docs:** `~/Code/ml-fluency/05-frontier-papers/README.md` (ordered list), `~/Code/ml-fluency/0*/README.md`, `~/Code/nanogpt-mlx/ROADMAP.md` (which rung is he on?), `research/_canon.md`.
- **Vault:** grep `research/`, `ideas/`, `projects/` for the core concepts. Does this **confirm** something he's written, or **contradict** it? A contradiction is worth more — lead with it.
- **Memory:** `~/.claude/projects/-Users-sameer-Desktop-knowledge/memory/`.
- **Code:** if there's a plausible touchpoint, **read the relevant part of the repo** and name the file and function. An unverified connection is a guess and reads like one.

**Finding nothing is a real, reportable result.** Most papers won't touch his work. Say so by *omitting the section*, not by inventing a link.

---

## Step 4: Write it

Path: `research/ai-ml/papers/<tag>.md`. Short memorable `tag` (`muon-scalable`, `react-agents`). Check it doesn't exist; don't overwrite. Non-AI/ML → `research/<domain>/papers/`.

### TRIAGE mode

```markdown
# <Title>
**Authors:** <authors> · **<arXiv id>**, <date> · [<url>](<url>)
**Category:** `<category>` · **Lane:** `<lane>`
**Source:** `~/.cache/knowledge/arxiv/<id>/` (entrypoint `<file>.tex`)
**Summarized:** <YYYY-MM-DD> — *by Claude, from the LaTeX source. Sameer has not read this yet.*

## Verdict

**<SKIP | SKIM | READ CLOSELY>** · lane: **<IMPLEMENT | CANON | FLUENCY | FUEL | EXPLORE | AWARENESS>**

<Why — grounded in the lane. What reading it buys him, and against what it costs.
If SKIP: the one sentence he needs, and he's done.
If SKIM: which sections, and what to look for.
If READ CLOSELY: what the hard part is, what to watch going in.>

## What it claims
<The real argument, on the paper's terms. Specific — real numbers, real mechanisms, the
equation if the equation is the point. This has to be *correct*; he may cite it.>

## What's actually new
<The grain of salt. Contribution vs. framing. What do the ablations quietly reveal? Which
claims does the evidence support and which are asserted? If the paper is honestly strong,
say so — manufactured skepticism is as useless as credulity.>

## The take
<One paragraph he could defend out loud. A *position*, not a re-summary. This is the
translation layer — the reason the skill exists.>

## Where it touches your work
<ONLY if Step 3 found something real. Name the file, the rung, the note. Contradictions first.
DELETE THIS SECTION ENTIRELY if there's no honest connection.>

## My take
<!-- Sameer's, after reading. Blank on purpose. -->
```

### MAP mode (canon)

No verdict — he's reading it. Give him the guide.

```markdown
# <Title>
**Authors:** <authors> · **<arXiv id>**, <date> · [<url>](<url>)
**Category:** `<category>` · **Lane:** `<lane>`
**Source:** `~/.cache/knowledge/arxiv/<id>/` (entrypoint `<file>.tex`)
**Mapped:** <YYYY-MM-DD> — *reading map by Claude. Read the paper; this is the map, not the territory.*

## Why this one is canon
<What it changed. What everything downstream inherits from it. Why not having read it is a hole.>

## The argument in one pass
<The spine of the paper — enough to orient, not to substitute.>

## Read closely / skim / skip, section by section
<Concrete. "§3.2 is the paper — read it twice. §4 is dated benchmark tables, skim.
Appendix A is the thing people actually cite."">

## What's aged, and what everyone misquotes
<The most valuable section. Which claims held, which didn't. What the paper is arguing
*against* that's now invisible because it won. Where the folk-understanding diverges
from what the text actually says.>

## Questions to hold while reading
<3-5 questions that make it an active read rather than a passive one. These should be
answerable from the text — check the answers exist before you write them.>

## Where it touches your work
<Same rule: only if real. Delete if not.>

## My take
<!-- Sameer's, after reading. Blank on purpose. -->
```

---

## Step 5: Update the index

Add or update the paper's row in `research/ai-ml/papers/_papers.md` — category, lane, verdict, status, one-line hook. **The index is the front door; a summary not in the index is a summary he'll never find.**

**Batch runs:** if you're one of several agents running in parallel, do **not** write the index — eight agents editing one file will race and clobber each other. Write only your paper file and return your row to the orchestrator.

If he reads it and it's durable, it graduates to `research/_canon.md`. That's his call, not yours — but say so if it's obviously canon-grade.

---

## Step 6: Hand it off in one line

Verdict (or "mapped") plus the single most interesting thing. Two sentences. Not a recap:

> "SKIM, fluency lane. Result is real but it's a constant-factor win, not a scaling-law win — the fitted exponents are identical and they don't mention it. `research/ai-ml/papers/muon-scalable.md`."

Then stop. Don't offer to go deeper. He'll say.

---

## Rules

- **The verdict names a lane.** A verdict with no lane is an opinion; a verdict with a lane is a decision. If no lane fits, that's usually a SKIP — say so.
- **Most papers are SKIM or SKIP.** A skill that says READ CLOSELY every time says nothing. His attention is the budget you're spending.
- **Never pad a section to fill the template.** Omit "Where it touches your work" when there's no connection. A forced link is how this rots into slop he skims past.
- **Read the source before you judge it.** Not the abstract, not your memory of the paper, not what other papers say about it. If you can't get the text, say so plainly.
- **The summary is Claude's; the take is his.** Leave `## My take` empty. Never fill it, never "help" by drafting it.
- **A summary routes the read — it does not replace it.** He offloads building, never learning. If the verdict is READ CLOSELY, he reads it; you gave him the map.
- **Source is cached, never committed.** `~/.cache/knowledge/arxiv/` — derived, re-downloadable. The vault holds what he thought, not what he downloaded.

## What this skill is NOT

- Not `/learn` — topic-level teaching, no specific source.
- Not `/compile` — that distills *his* notes into writing. This produces a reference note from a source he hasn't read yet.
- Not a substitute for reading.
