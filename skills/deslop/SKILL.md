---
name: deslop
description: Use when prose reads machine-written and needs to sound human — blog posts, docs, READMEs, emails, PR descriptions, landing copy. Finds the specific phrases and structures that mark text as LLM output and removes them without rewriting the author's voice. Invoke with /deslop <file-or-text>.
allowed-tools: Read, Write, Edit, Glob, Grep, Bash
---

# Deslop: $ARGUMENTS

Strip the tells that mark prose as machine-written. Keep the author's voice.

## Iron Law

**Delete the tell, keep the sentence.** This is an edit, not a rewrite.

The failure mode is not missing a tell — it is "fixing" the piece into your own
voice, which is the same slop wearing a different coat. If a sentence has no tell
in it, it does not get touched, however you would have written it. Sameer's
argument, structure, jokes, and rhythm survive intact.

If more than about a third of the text needs changing, stop and say so. That is
not a de-slop job; the piece needs rewriting by its author, and doing it for them
loses the thing worth keeping.

## Step 1: Read the source

`$ARGUMENTS` is a file path, a glob, or pasted text. For a path or glob, Read
every match in full before editing anything — a tell is often only visible
against the paragraphs around it (three sections that each open the same way is
a tell; one section opening that way is a sentence).

If `$ARGUMENTS` is empty, ask what to de-slop. Do not guess at recently edited
files.

## Step 2: Scan against the pattern list

Read `references/patterns.md` and scan for each category. Do not work from
memory — the list exists because the tells drift and recall of them is exactly
what a model is worst at.

Record every hit as `file:line — "phrase" → replacement`. Build the whole list
before making a single edit, so you can apply Step 3's judgment across the piece
rather than one line at a time.

**Completion criterion:** you have scanned for every category in
`patterns.md`, not just the vocabulary one. Structural and rhythm tells are the
higher-signal half and the easier half to skip.

## Step 3: Judge each hit before applying it

A pattern match is a candidate, not a verdict. Drop the hit when:

- **The word is load-bearing.** "Delve" in a piece about mining. "Robust" as a
  term of art in statistics. The list flags phrasing that carries no information;
  where it carries information it stays.
- **It is a quotation.** Never edit inside quoted text, a transcript, a citation,
  or a code block. Quoting someone who wrote "leverage" is reporting, not sloppy.
- **It is the author's actual habit.** If the same construction appears in older
  writing that predates the piece, it is a voice, not a tell. Check with
  `git log -p` on the file, or another file in the same directory, when unsure.

Whatever survives this step gets applied with Edit, one phrase at a time.

## Step 4: Read it aloud, then check the rhythm

Vocabulary edits are the easy half; the giveaway that survives them is cadence.
After applying the edits, reread the piece for:

- Paragraphs that are all the same length.
- Sentences that are all medium-length, with no fragment and no long one.
- Every section opening with the same move (a question, a definition, a "when it
  comes to X").
- A summary paragraph that restates what the reader just read.

Fix these by cutting, not by adding. The last one is almost always deletable
whole.

## Optional: plain-English mode

Off by default. Runs only when Sameer asks for it — "plain English", "simpler",
"--plain" — because it relaxes the Iron Law for *register* while keeping it for
content. Borrowed from [claudish-to-english](https://github.com/gvzdv/claudish-to-english),
whose entire method is the two rules below.

After Steps 2–4, make one more pass that rewrites for a reader in a hurry:

- Short sentences. Everyday words. If a simpler word says the same thing, use it.
- **The preservation contract:** keep every fact, name, number, link, and file
  path exactly. Fenced code blocks, YAML frontmatter, and quotations are
  reproduced untouched. Markdown structure (headings, lists, tables) stays.

This mode may reword sentences that have no tell in them — that is the point of
opting in. It still may not cut claims, reorder the argument, or add anything.
Report it as its own row group in Step 5 so the two kinds of change stay
distinguishable.

## Step 5: Report

Show a table of what changed and why:

| Location | Was | Now | Tell |
|---|---|---|---|

Then state, explicitly:

- **What you left alone and why** — every hit dropped in Step 3. This is the
  most useful part of the report, because it is where you might have been wrong,
  and Sameer can overrule you in one line.
- **Whether the rhythm pass changed anything structural** (a cut paragraph, a
  merged section), since that is a bigger intervention than a word swap.

End with `DONE` or `DONE_WITH_CONCERNS` per `_conventions.md`. Use
`DONE_WITH_CONCERNS` when you hit the one-third rule in the Iron Law, or when a
tell is load-bearing enough that removing it would change the meaning and you
left it in.

## Notes

- The related instrument inside **hunt** (`src/lib/checks/ai-tell.ts`) is a
  deterministic regex audit over résumés and cover letters, with a narrower,
  résumé-tuned list. Deliberately separate: it runs in a product UI on every
  keystroke and cannot exercise Step 3's judgment. If a pattern proves itself
  here and is objective enough to regex, it is worth porting there by hand.
- Report the reading as "this pattern-matches LLM boilerplate", never "this is
  AI-generated". The first is checkable; the second is a claim about the writer
  that no one can verify.
