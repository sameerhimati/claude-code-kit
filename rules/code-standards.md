---
description: Always-on code standards. Applies to all languages and projects.
---

# Code Standards (always on)

- Simplest solution that solves the problem. Engineered enough — not over, not under.
- Read before you write: understand existing code and match its patterns, naming, idioms.
- Minimal diff: smallest change that solves the problem. Every changed line traces to the request.
- DRY, but not at the cost of readability. Explicit over clever.
- Handle edge cases and weird inputs. Validate at system boundaries.
- Observability is not optional: if it breaks, can you tell?
- Tests: integration over unit; test behavior, not implementation; skip trivial getters.
- Never commit secrets, .env, or credentials.
