---
name: compile
description: >
  Distill reading into durable artifacts. Two source modes: (A) raw files in inbox/raw/
  or inbox/raw/preserved/, (B) reading-notes entries in research/reading-notes/. Default
  output is a post draft in writing/; atomic notes in research/ or ideas/ are the
  exception, reserved for high-bar claims. Conversational. Invoke manually with /compile
  or when asked to "distill this", "compile my reading notes", "turn this into a post",
  "process raw sources", or "what's ready to distill".
disable-model-invocation: false
allowed-tools: Bash, Read, Write, Edit, Glob, Grep, AskUserQuestion
---

# Compile: $ARGUMENTS

You are distilling Sameer's reading into durable artifacts. The pipeline: **source → discuss → distill → cross-reference → log.**

Two source modes:
- **Mode A — raw file.** A discrete document in `inbox/raw/` or `inbox/raw/preserved/` (book excerpt, transcript, PDF). Old pipeline: raw → atomic note → delete raw.
- **Mode B — reading-notes entry.** An entry Sameer filled into `research/reading-notes/<cluster>.md` after reading a URL-based source. New pipeline: reading-notes entry → post draft (default) or atomic note (exception).

**Default output = post draft.** Most distillation should produce writing, not atomic notes. Atomic notes are reserved for high-bar claims (see Step 3).

---

## Step 0: Pick the Source

If `$ARGUMENTS` specifies a file or entry, use that. Otherwise:

1. **List Mode A sources:** everything in `inbox/raw/` (exclude README.md, preserved/, and any clips/ subfolder)
2. **List Mode B sources:** entries in `research/reading-notes/*.md` that look distillable — recent entries, entries with "Extract candidates" filled in, entries without a matching atomic note yet
3. Show Sameer both sets, briefly — title, date, one-line shape ("Mode A: raw clip about X" or "Mode B: reading-notes entry on Y, filled in Tuesday")
4. Ask: "Which one do you want to distill? Or should we go through them in order?"

If batch mode ("compile everything"): process one at a time, confirm each before moving to the next. Mode A raw files get deleted after successful compile; Mode B reading-notes entries stay as working history.

---

## Step 1: Read and Engage

This is a learning conversation, not a filing task.

1. Read the full raw source silently
2. **Don't summarize yet.** Instead, ask Sameer to read it first (or confirm he already has):
   - "Have you read this one? Take a few minutes if not — I want your take before I give mine."
3. Once he's read it, **ask him questions first:**
   - "What's the core claim here?"
   - "What surprised you?"
   - "How does this connect to anything you're already working on?"
   - "Do you agree with the author? Where would you push back?"
4. **Then** share your analysis:
   - What the source actually says (correct any misreadings gently)
   - What's new vs what's already in the vault
   - Connections Sameer might have missed
   - Where the author might be wrong or oversimplifying
5. **Quiz on key concepts** if the source is technical:
   - "Can you explain [concept] back to me in your own words?"
   - "If you had to apply this to [project], how would you?"
   - "What's the difference between [thing A] and [thing B] from this article?"

The goal: Sameer should understand the material deeply enough to explain it to someone else. If he can't, dig deeper before moving on.

**Only proceed to Step 2 when Sameer has genuinely engaged with the source.** Filing without understanding is just hoarding.

---

## Step 2: Decide the Output Type

**Default: post draft.** Most compile sessions should produce a post, not an atomic note. Propose a post draft at `writing/<slug>.md` (which is symlinked to `~/Desktop/code/sameerhimati.github.io/content/posts/`) with frontmatter, `draft: true`, and structure.

**Exception: atomic note.** Reach for an atomic note only when the claim clears one of these bars:
1. **Reference from multiple places** — the claim belongs in 3+ existing or planned notes, so it needs to exist standalone
2. **Cross-source pattern** — a synthesis across multiple articles, not a single article's point
3. **Decision-changing** — a conviction that shapes how Sameer builds Atlas, Keepr, Holding
4. **Durable, not topical** — survives the specific article you read it in

If unsure, default to post draft. Atomic notes are the exception, not the rule.

### Destinations
- **`writing/<slug>.md`** — post draft (DEFAULT)
- **`research/<domain>/<topic>.md`** — atomic note for high-bar topic learnings
- **`ideas/<claim-as-filename>.md`** — atomic note for high-bar cross-domain patterns
- **`projects/<project>/intuitions.md` or `projects/<project>/decision-*.md`** — if it's directly changing a project's direction (append to intuitions) or locking a decision (new file)

**For atomic notes:** check if a relevant note already exists — update rather than create. Link from the domain's learning map (`research/<domain>/_subject.md`) if one exists.

Present the proposal: "I'd draft this as a post at `writing/mcp-first-principles.md`, structured around the three insights you flagged. Sound right?" OR "This clears the atomic note bar because [reason] — I'd put it at `ideas/workflows-before-agents.md`. Sound right?"

---

## Step 3: Write the Output

### For post drafts (default path)

Write the draft in Sameer's voice as much as you can infer from his reading-notes, but **do not ghostwrite** personal-voice pieces. For Building/Learnings tagged posts, drafting is fine. For Reflections tagged posts, scaffold only — leave the prose to him.

Hugo frontmatter format:
```yaml
---
title: "<Title>"
date: "YYYY-MM-DD"
description: "<one-line description>"
tags: ["Building" | "Learnings" | "Papers" | "Books" | "Reflections"]
draft: true
---
```

Structure the draft around the insights Sameer surfaced in the reading-notes entry. Don't summarize the source — use the source to support claims he's making. Cite the source URL at the bottom.

Start with `draft: true` always. Sameer flips it when he's ready to ship.

### For atomic notes (exception path)

Follow vault conventions:

- **Filename is a claim** (for ideas) or **descriptive** (for research topics)
- **Atomic**: one idea per file, 15-40 lines
- **Your own words**: not a copy of the source — what SAMEER understands
- **Structure**:
  ```
  # Title (the claim, spelled out)

  Core claim in 1-2 sentences.

  ## Why this matters

  ## How I'd apply this

  ---

  **Source:** [article/paper/thread], [author], [URL]
  **See also:** [[wikilinks to related notes]]
  ```

For longer or richer sources that clear the atomic bar on multiple independent points, you may create multiple notes — one per claim. Ask first: "I see 3 separate claims that could each be atomic. Want them as one note, three, or a post that covers all three?"

---

## Step 4: Cross-Reference

After writing the compiled note:

1. **Update the learning map** if one exists for the domain — add a wikilink to the new note
2. **Add wikilinks** to and from related existing notes
3. **Update `index.md`** if the note changes routing (new domain, new idea worth listing)
4. **Check for contradictions** — does the new source disagree with anything already in the vault? Flag it.

---

## Step 5: What Changes?

The compiled note exists. Now close the loop. Ask Sameer:

**"Does this change a skill, a project, or a belief?"**

Walk through each:

1. **Skill** — Does this change how we work? A new pattern, a better process, a tool we should use differently?
   - → Update the relevant skill in `~/.claude/skills/`, or create a new one if the pattern is significant enough.
   - Example: learning about eval-driven development → update `/feature-flow` to include an eval step.

2. **Project** — Does this change how a specific project should be built, architected, or prioritized?
   - → Update the project's `CLAUDE.md`, a decision note in `projects/<project>/`, or the vault's project dashboard.
   - Example: managed agents article → update Atlas architecture notes, add to Holding roadmap.

3. **Belief** — Does this change a conviction that shapes decisions across projects?
   - → The compiled note itself IS the artifact. Make sure it's wikilinked well so it surfaces when relevant.
   - Example: "push-based agents beat dashboards" → conviction that shapes Keepr, Holding, Minions design.

If the answer is **none of the above**, that's fine — it's a reference note. But name that explicitly: "This is reference material, nothing changes right now."

If something DOES change, **make the change right now** — don't defer it. The insight is freshest in this moment. Update the skill, the CLAUDE.md, or the project note before moving on.

---

## Step 6: Log It

Append to `log.md`:

```
## [YYYY-MM-DD] compile | [Source title]
Compiled `inbox/raw/[filename]` → `[destination path]`. Key insight: [one line].
Changed: [skill/project/belief/nothing] — [one line describing what was updated, or "reference only"]
```

---

## Rules

- **Never skip the discussion.** The value is in Sameer's reaction to the source, not just the source itself.
- **Don't copy-paste from sources.** Outputs are in Sameer's voice — what he understood, what matters to him.
- **One source/entry at a time.** Even in batch mode, discuss each before distilling.
- **Mode A (raw files): delete after compile.** Once a raw source is distilled into an artifact, delete the raw file from `inbox/raw/`. The output's frontmatter URL is the reference. Don't hoard receipts. **Exception:** files in `inbox/raw/preserved/` stay — they're the permanent non-URL exception (books, transcripts).
- **Mode B (reading-notes entries): keep as working history.** Don't delete the entry. Append a `> **Distilled:** <date> → <output path>` line at the bottom so future-you can see it's been processed.
- **Default to post drafts.** Atomic notes are the exception. If Sameer keeps saying "this is more of a post," that's a signal the atomic bar is too low — trust the pattern.
- **Respect voice boundaries.** Building/Learnings/Papers/Books posts can be drafted by you. Reflections posts cannot — scaffold only. See `feedback_writing_voice` memory.
- **If the source is noise** — say so. "This doesn't add anything new to the vault. Skip it?" is a valid output.
- **If the source sparks a project idea** — capture it in `projects/<project>/intuitions.md`, not `ideas/`.

## Reading-notes specific: the Application section

If a Mode B entry has an `### Application` section (with Apply now / Apply later / Skip bucketing), read it and route each item during Step 5:

- **"Apply now"** items → make the change immediately. Update the skill, edit the project file, commit the config. Don't defer.
- **"Apply later"** items → move to their real destination based on the trigger. "When building Clinic MCP" → append to `projects/clinic/` or `research/ai-ml/mcp-course-notes.md`. "When Keepr integrates Plaid" → append to `projects/keepr/intuitions.md`. The reading-notes entry keeps a copy as history, but the actionable version lives where future-you will hit it.
- **"Skip"** items → stay in reading-notes, no action.

If the Application section is empty or missing, ask the "what changes: skill/project/belief?" question in Step 5 as usual. The section is optional — it's just a nice pre-population when Sameer chose to use it.
