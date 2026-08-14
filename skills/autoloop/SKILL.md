---
name: autoloop
description: Run an autonomous, self-correcting loop on a task by first establishing a verifiable success condition, then iterating until it passes. Use for longer autonomous work where Claude should fix its own mistakes — especially when no test exists yet. Invoke with /autoloop. This is the front-end to /goal.
disable-model-invocation: false
allowed-tools: Read, Glob, Grep, Bash, Edit, Write, Agent
---

# Autoloop

A verifiable domain is what lets a session run long and self-correct. This skill makes the goal verifiable FIRST, then loops to green. The hard part `/goal` assumes you've already done — defining the check — is Step 1 here.

## Step 1: Define the goal and acceptance criteria
State, in one or two sentences, what "done" means in observable terms. Not "improve X" but "X returns Y for input Z", "endpoint returns 200 matching schema S", "page renders with no console errors".

## Step 2: Establish the verifier (the key step)
Find or create a check that exits 0 on success, non-zero on failure:
- Prefer an existing test/command: `pytest path::test`, `npm test`, `tsc --noEmit`, `npm run build`, a curl smoke check.
- If none exists, WRITE one — usually a failing test that encodes the acceptance criteria. A custom script is fine when no framework fits.
- Confirm it RUNS and currently FAILS (red). A loop without a red→green signal is not an autoloop — escalate if you can't produce one.

## Step 2b: Align before looping (the one human gate)
The loop then runs long and autonomously, so confirm direction NOW, not after. Present in plan mode: the goal, the acceptance criteria, and the exact verifier you'll loop against. Get sign-off (ExitPlanMode) before Step 3. If the user corrects the scope or the check, fix it here — grinding toward the wrong target is the costliest failure.

## Step 3: Loop to green
Implement → run the verifier → read the failure → fix → repeat. Because the verifier is objective, you may iterate well past the usual 3-attempt cap **as long as each iteration moves the check closer to passing**. Stop and escalate if you see the same failure twice with no new information, or if the only way to "pass" is to weaken the test.

**Iteration cap by verifier cost:** when the verifier is remote CI (each attempt costs a push), cap at 3 fix iterations. Local verifiers may exceed 3 while strictly converging. Either way, stop on a repeated identical failure.

For long unattended runs, hand the verified condition to `/goal` (loop-until-condition), or pair with auto mode + a Stop hook so CI is the verifier.

## Step 4: Confirm and report
On green, run the FULL suite (lint, type-check, tests, build) — not just the one check — so a local pass doesn't hide a global break. Show command output as evidence.

**Residual exit (if you stop short of green):** file residuals — write each unresolved failure (symptom, root-cause hypothesis, what was tried) into the final report so the next session inherits durable state. BLOCKED with filed residuals beats BLOCKED with a shrug.

**Compound write-back:** if the loop revealed a reusable lesson, propose it to the appropriate config home (project CLAUDE.md / rule / skill), deletion-tested. Ask before writing to `~/.claude`.

## Iron law
No loop without a verifier. Define red before you chase green. Never edit the test to make it pass unless the test itself was wrong — and say so explicitly.

End with a completion status (DONE / DONE_WITH_CONCERNS / BLOCKED / NEEDS_CONTEXT).
