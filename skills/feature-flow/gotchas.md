# Feature Flow Gotchas

Build this up over time as failure patterns emerge.

## Known Issues
- Feature-flow was never triggered automatically before March 2026 because the description said WHAT it does, not WHEN to use it. Fixed with new description.
- The old version referenced a "reviewer agent" that no longer exists. Now uses gstack /review.
- Step 4 (verify) was often skipped when features seemed "trivial." Never skip — trivial features break builds too.

## Common Failure Patterns
- **Scope creep**: Feature starts small, grows during implementation. The scope check in Step 1 exists for a reason — if you can't describe it in one commit message, split it.
- **Test-after syndrome**: Writing tests after implementation leads to tests that verify the code works as written, not as intended. Write tests alongside.
- **Missing edge cases**: Step 2 asks about edge cases. Actually think about them — null inputs, empty arrays, concurrent access, auth boundaries.
