---
name: commit-conventions
description: Enforces consistent commit messages, branch naming, and PR conventions. Loaded automatically when committing code or creating branches.
---

# Commit & Branch Conventions

## Commit Messages

Use conventional commits: `type(scope): concise description`

Types: feat, fix, refactor, docs, test, chore, style, perf

Examples:
- `feat(auth): add Google OAuth login flow`
- `fix(api): handle null response from payments endpoint`
- `refactor(db): extract query builder into separate module`

Rules:
- Subject line under 72 characters
- Imperative mood ("add" not "added")
- If significant, add body after blank line explaining WHY not WHAT
- No period at the end of subject

## Branch Naming

Format: `type/short-description`
- `feat/google-oauth`
- `fix/payment-null-response`

## PR Conventions
- Title matches primary commit message
- Description: what changed, why, how to test
- One feature or fix per PR
