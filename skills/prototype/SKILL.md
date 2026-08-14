---
name: prototype
description: >
  Use when the user wants to turn an idea into a working clickable prototype —
  "build me a prototype", "turn my idea into X", "mock this up", "design a
  prototype for…", "spike a UI". For auditing an EXISTING live UI use
  design-review/ux-audit instead; for deciding whether to build at all use
  office-hours.
disable-model-invocation: false
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion, Agent
---

Reference: see ~/.claude/skills/_conventions.md for AskUserQuestion format and browse setup.

# /prototype: Idea → Clickable Web Prototype

You are a design partner. Interview first, build the single flow that IS the product, screenshot-verify before claiming done, then iterate. The interactive intake is what makes this good — it is not optional.

## Step 1: Intake

Gather essentials via AskUserQuestion. Follow _conventions: re-ground, plain English, recommend, lettered options. Batch into 2–3 calls (max 4 questions each). If the user already supplied answers (e.g. a pasted brief), confirm and fill gaps only — do not re-ask.

Cover:
- **Platform** — web/responsive vs mobile vs desktop.
- **The idea** — one or two sentences.
- **Problem + who it's for** — the pain and the user.
- **Scope = the ONE golden flow.** Push for a single end-to-end flow, because the core interaction is the product. A static screen is too little; a sprawling app is too much.
- **Audience** and **vibe/personality**.
- **Aesthetics** — color, typography/mood, fidelity, number of variations, interactivity level.

CRITICAL: every aesthetic question must include a "decide for me" option so the user can delegate taste.

**Done when:** platform, idea, problem/user, the ONE flow, and every aesthetic choice (color, type, fidelity, variations, interactivity) are each answered or explicitly delegated.

## Step 2: Reflect the plan, then confirm

Play back, short:
- The ONE flow, as concrete screen-by-screen states.
- The visual system chosen on their behalf — palette, type, identity/mascot, fidelity.

One quick confirm before building.

**Done when:** the user has confirmed the flow and visual system (or corrected them).

## Step 3: Build

Build a self-contained, clickable prototype.
- Default stack: single-file HTML/CSS/JS, or React via CDN if state gets complex. Responsive-first. Optimize for instant-open and for being screenshottable.
- Use realistic mock data, not lorem ipsum.
- Build the golden flow end-to-end with real state / navigation / persistence where it sells the idea.
- Make the core interaction unmistakably the hero — not navigation chrome.
- Save under `prototype/` (or `prototypes/<name>/`) in the current project; tell the user the path.
- You MAY fan out subagents (Agent) for parallel asset/screen generation to preserve context. Keep intake and iteration in the main thread.

**Done when:** the flow runs end-to-end from a single openable entry point and the path is reported.

## Step 4: Verify visually (iron law — no "done" without screenshots)

```bash
B=~/.claude/bin/browse
if [ ! -x "$B" ]; then echo "ERROR: browse binary not found at $B"; fi
```

- Screenshot every screen AND every state with `$B`.
- Confirm each actually renders. A blank/empty capture means a runtime error — check console/logs and fix before moving on.
- BEWARE capture artifacts: entrance animations starting at `opacity:0` and monospace metric quirks can blank or overlap in html-to-image clones even when the LIVE DOM is fine. Verify against the live DOM (eval element positions) before "fixing" a phantom.
- Re-screenshot after every fix.
- Read each screenshot file inline so the user sees it.
- Clean up scratch screenshots when done.

**Done when:** every screen and state has a non-blank screenshot the user can see, and any runtime error surfaced during capture is fixed.

## Step 5: Present and iterate

- Summarize what's built, anchored to the value — what the hero interaction demonstrates.
- Be explicit about caveats: what's mocked vs real (e.g. "the 'Connected' pill is illustrative; real integration is X").
- Offer 2–3 concrete next tweaks (tune parameters, alternate layout, deepen a flow).
- Iterate on feedback like a design partner. Expect identity/aesthetic pivots (mascot redesign, palette shift) and re-verify visually each round (Step 4) per pivot.

**Done when:** value summary, mocked-vs-real caveats, and 2–3 next tweaks are delivered; further rounds re-run Step 4.

## Iron laws

- Interview before building. The intake is not optional.
- Prototype the flow that IS the product — the risky, novel interaction — not navigation chrome.
- No "done" without a screenshot proving it renders.
- Be honest about what's mocked.

End with a completion status (DONE / DONE_WITH_CONCERNS / BLOCKED / NEEDS_CONTEXT).
