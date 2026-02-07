---
name: feature-flow
description: Build a single feature end-to-end. One feature = one commit. Plan, implement, test, review, commit.
disable-model-invocation: true
allowed-tools: Task, Bash, Read, Write, Edit, Glob, Grep
---

# Feature: $ARGUMENTS

## Step 1: Scope Check
- Is this ONE feature or multiple? If multiple, stop and split.
- Can this be one commit message? If not, it's too big.

## Step 2: Plan
- Which files change?
- New files needed?
- Data model changes?
- Edge cases?

Present plan. Wait for confirmation.

## Step 3: Implement
- Follow existing codebase patterns
- Don't touch unrelated code (note for /techdebt)
- Comments only for non-obvious logic

## Step 4: Verify
- Run existing tests
- Write new tests for the feature
- Run linting/type-checking

## Step 5: Review
Spawn reviewer agent. Address any 🔴 Critical items.

## Step 6: Commit
- Stage ONLY files for this feature
- Commit message per commit-conventions skill
- Report: ✅ Feature complete, commit hash, files changed, test status

## Anti-patterns
- ❌ "While I'm in this file, let me also fix..." — NO. Note it, move on.
- ❌ Committing unrelated changes together — NEVER.
- ❌ Skipping review — ALWAYS review, even for small changes.
