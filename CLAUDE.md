# Project Guidelines

## Coding discipline (always apply)

Adapted from the Karpathy LLM-coding guidelines. Bias toward caution over speed; use judgment on trivial tasks.

- **Think before coding.** State assumptions explicitly; if uncertain, ask. If multiple interpretations exist, surface them instead of choosing silently. If a simpler approach exists, say so.
- **Simplicity first.** Write the minimum code that solves the problem — no speculative features, no abstractions for single-use code, no unrequested configurability, no error handling for impossible cases. If 200 lines could be 50, rewrite. Test: would a senior engineer call this overcomplicated?
- **Surgical changes.** Touch only what the request requires; every changed line should trace to it. Don't refactor, reformat, or "improve" adjacent code. Match existing style. Remove only the orphans your own changes create; flag pre-existing dead code instead of deleting it.
- **Goal-driven execution.** Turn tasks into verifiable criteria and loop until they pass (e.g. "fix the bug" → write a failing test that reproduces it, then make it pass). For multi-step work, state a brief plan with a verify step per item.

Full version: [.claude/skills/karpathy-guidelines/](.claude/skills/karpathy-guidelines/SKILL.md)
