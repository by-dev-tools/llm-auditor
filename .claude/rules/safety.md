---
paths:
  - "agents/auditor.md"
  - "skills/audit-plan/SKILL.md"
  - "skills/audit-completion/SKILL.md"
  - "scripts/extract_session.py"
  - "scripts/bounding_logic.py"
  - ".claude-plugin/plugin.json"
  - ".claude-plugin/marketplace.json"
  - "evals/run_evals.py"
  - "evals/ground_truth.yaml"
---

# Safety-Critical Code Rules

This file loads automatically when you touch the plugin's published artifacts or its eval harness. These are the files that determine what the auditor does, what it sees, and how we know whether it's right.

Safety-critical surfaces in this plugin:

- **`agents/auditor.md`** -- the auditor's system prompt. Changes here directly alter audit behavior. Small wording shifts can flip false-positive rates.
- **`scripts/extract_session.py`** and **`scripts/bounding_logic.py`** -- session parsing and context assembly. Silent failure modes here starve the auditor of evidence without surfacing an error. Already known to skip malformed JSONL lines silently (line ~90); be deliberate about any change to that.
- **`skills/audit-plan/SKILL.md`** and **`skills/audit-completion/SKILL.md`** -- slash command dispatch. Shell substitution is used to inject preprocessed context; no size guards today.
- **`.claude-plugin/plugin.json`** and **`.claude-plugin/marketplace.json`** -- install surface. Breaking these breaks installation for every user.
- **`evals/run_evals.py`** and **`evals/ground_truth.yaml`** -- the only regression signal we have. If evals break silently, we lose the ability to detect auditor behavior drift.

## Before modifying safety-critical code

1. Run `git log --oneline -5 -- <file>` to check for recent deliberate safety decisions.
2. If a recent commit mentions "crash", "data loss", "safety", "integrity", or "fallback" -- read that commit's diff to understand what was deliberately added.
3. Preserve safety behavior through rewrites. If restructuring a function, verify all safety-critical paths from the previous version still exist.

## When committing safety changes

- Flag the change in the commit message with a `SAFETY` marker.
- Flag the change in `core-docs/history.md` with a `SAFETY` marker.
- Explain what safety behavior was preserved, modified, or added.

## Never silently downgrade error handling

- Don't replace explicit error handling with silent fallbacks (e.g., `fatalError` to silent catch, `throw` to `try?`, error alerts to console logs).
- Don't convert user-facing warnings to debug-only logging.
- Don't remove validation without documenting why it's no longer needed.
