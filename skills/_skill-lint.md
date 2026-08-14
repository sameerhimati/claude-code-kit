# Skill Lint

The 7-point test for every skill. Single source of truth — referenced by `config-audit` and the `skill-author` agent, not copied into them. This is a reference doc, not a skill.

Run it per skill; each point is pass/fail. Bias toward removal — the cost of a weak line is paid every session it loads.

1. **No-op test** — does each line change behaviour vs the model's default? If the model already does it by default, cut it. (This is the operational definition of the deletion test.)
2. **Duplication test** — does any meaning appear in more than one source of truth? Watch description↔body identity overlap, synonyms that rename one branch, and facts that also live in a rule or CLAUDE.md. One trigger per branch.
3. **Sediment test** — is every line still live and current? Flag stale paths, dead skill refs, and instructions for removed tools.
4. **Sprawl test** — is SKILL.md too long regardless of staleness? Push heavy content behind pointers (`reference.md`); split steps that tempt rushing.
5. **Premature-completion test** — does each step have a CHECKABLE completion criterion, and an EXHAUSTIVE one where it matters? If a step is irreducibly fuzzy, split off the post-completion work into its own step.
6. **Invocation/cost test** — is `disable-model-invocation` set correctly? Model-invoked only if an agent or another skill must reach it autonomously (costs context load every session); otherwise user-invoked (zero context load, costs Sameer's memory to recall). Flag model-invoked skills with zero usage history as flip candidates.
7. **Description-as-trigger test** — does the description front-load the leading word and read as a trigger ("Use when…"), not a summary?

Derived from Matt Pocock's writing-great-skills rubric (see memory `ai-dev-loop-references`).
