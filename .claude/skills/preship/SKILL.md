---
name: audit
description: >
  Review recent changes against project docs and standards. Use after completing
  a feature or before shipping to catch gaps. Checks code against spec, design
  language, and documented feedback rules.
context: fork
agent: Explore
allowed-tools: Read, Grep, Glob, Bash
---

You are auditing recent changes against project standards. This runs in an isolated context.

## 1. Gather context

- Run `git diff main..HEAD --name-only` to list changed files
- Run `git log --oneline main..HEAD` for commit history
- Read `CLAUDE.md` for project standards
- Read `core-docs/feedback.md` for documented rules and past corrections

## 2. Check documentation completeness

- Does `core-docs/history.md` have entries for the changes? Are they complete (what, why, tradeoffs)?
- Does `core-docs/plan.md` reflect the current state?
- Were any user corrections made that aren't captured in `core-docs/feedback.md`?

## 3. Check against feedback rules

For each entry in `core-docs/feedback.md`, check if the recent changes violate any synthesized rules. Flag violations.

## 4. Check design language compliance (if UI changes)

- Read `core-docs/design-language.md`
- Scan changed UI files for hardcoded values that should use design tokens
- Check accessibility basics (labels, contrast, keyboard access)

## 5. Report

Produce a concise report:

```
## Audit Results

### Documentation
- [pass/fail] history.md updated
- [pass/fail] plan.md reflects current state
- [pass/fail] feedback.md captures corrections

### Feedback Rules
- [list any violations of synthesized rules]

### Design Language (if applicable)
- [list any hardcoded values or missing tokens]

### Recommendations
- [actionable items to fix before shipping]
```
