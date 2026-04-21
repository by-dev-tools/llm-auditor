# CLAUDE.md -- Assumption Auditor

## What This Is

A Claude Code plugin that audits a session for unverified assumptions, diagnoses, and completion claims. After Claude proposes a plan or declares work done, the plugin reads the session transcript, identifies load-bearing claims that lack supporting evidence, and surfaces them in a fixed output format. The plugin does not verify anything itself -- it only flags gaps.

**Core thesis:** the most expensive errors Claude makes in a session are confident claims that were never actually checked. A passive, skeptical auditor run between "plan" and "execute" (or between "done" and "trust") catches those claims cheaply, before they compound.

## Repository Layout -- Read This First

This repo contains **two distinct surfaces**. Don't confuse them.

### Plugin artifacts (what ships to users)

These files are published when the plugin is installed. They live at repo root because Claude Code's plugin convention requires it.

| Path | Purpose |
|------|---------|
| `.claude-plugin/plugin.json` | Plugin manifest |
| `.claude-plugin/marketplace.json` | Marketplace metadata |
| `agents/auditor.md` | Auditor subagent system prompt |
| `skills/audit-plan/SKILL.md` | `/audit-plan` slash command |
| `skills/audit-completion/SKILL.md` | `/audit-completion` slash command |
| `scripts/extract_session.py` | Session preprocessing |
| `scripts/bounding_logic.py` | User-message windowing |
| `evals/` | Regression fixtures and harness |
| `README.md` | User-facing install/use docs |
| `DISAGREE.md` | Feedback log for wrong audits |

### Project-dev infrastructure (how we build the plugin)

These files help Claude sessions develop and maintain the plugin. They are not published.

| Path | Purpose |
|------|---------|
| `CLAUDE.md` | This file -- always loaded |
| `core-docs/` | Living project knowledge (plan, history, feedback, spec, workflow) |
| `.claude/agents/` | Project-dev agents (planner, domain, testing, docs) |
| `.claude/skills/` | Project-dev workflows (`/ship`, `/preship`) |
| `.claude/rules/` | Auto-loading scoped rules |
| `.claude/settings.json` | Hooks (secret blocking) |
| `.context/` | Per-session scratch |

When you see `agents/` at root it's the **plugin's** auditor subagent. When you see `.claude/agents/` it's the **project-dev** roles for building the plugin. Same for `skills/` vs `.claude/skills/`.

## Tech Stack

- **Platform:** Claude Code plugin (distributed via `/plugin install`)
- **Language:** Python 3.7+ (stdlib only, no external deps) for preprocessing; Markdown for prompts and skills
- **AI:** delegates to a Claude Code subagent defined in `agents/auditor.md`; no direct API calls
- **Persistence:** none -- all state lives in the user's session files (`~/.claude/projects/<slug>/*.jsonl`)

## Product Principles

- **Passive over active.** The auditor reads; it does not re-run tools. Verification is the user's call.
- **Evidence or silence.** If a claim can't be challenged with specific evidence from the session, don't flag it. "No issues flagged" is a valid output.
- **Fixed output format.** Every audit returns the same structured shape so users and regression tests can parse it reliably.
- **Narrow scope beats wide scope.** Four categories (unverified diagnosis, completion, assumption, recall) -- not more.
- **Feedback loop is load-bearing.** `DISAGREE.md` is the source of prompt-tuning work and new eval cases. Treat user disagreements as data, not noise.

## Core Documents

All living project knowledge lives in `core-docs/`. Read before acting; update before shipping.

| Document | Path | Purpose |
|----------|------|---------|
| Plan | `core-docs/plan.md` | Current focus, active work, handoff notes |
| History | `core-docs/history.md` | Decision log -- what, why, tradeoffs, SHA |
| Feedback | `core-docs/feedback.md` | Synthesized user corrections (FB-XXXX) |
| Spec | `core-docs/spec.md` | Plugin scope, features, categories |
| Workflow | `core-docs/workflow.md` | Agent workflow and recipes |

## Agent Workflow

Project-dev agents live in `.claude/agents/`. Invoke via `claude --agent <name>` or by name in conversation. Use `/clear` between phases.

| Agent | Role | When to use |
|-------|------|-------------|
| `planner` | Scope features, write goals, update plan.md | Starting or refining work |
| `domain` | Python scripts, prompt changes, eval logic | Any code or prompt change |
| `testing` | Evals, fixtures, regression cases | After domain changes |
| `docs` | history.md, plan.md, commit | Shipping completed work |

Slash commands: `/ship` (push + PR + merge), `/preship` (standards review before shipping). Don't confuse these with the plugin's own `/audit-plan` and `/audit-completion` -- those are user-facing commands published by this plugin.

## How to Work

1. **Read before writing.** Check `core-docs/plan.md` for current focus and `core-docs/feedback.md` for past corrections.
2. **Respect the boundary.** Changes to plugin artifacts (root `agents/`, `skills/`, `scripts/`, `evals/`, `.claude-plugin/`, `README.md`) change user-visible behavior. Changes under `.claude/` and `core-docs/` do not.
3. **Prompt changes are code changes.** `agents/auditor.md` is the auditor. Treat edits like edits to a deployed service: write an eval fixture first, update `history.md`, tune deliberately.
4. **Follow the rules.** `.claude/rules/` auto-loads safety and documentation discipline when you touch matching files.

## Quality Bar

Code and prompts don't ship unless they meet these standards simultaneously:

- **Correct:** auditor outputs match the `ISSUE` / `AUDIT SUMMARY` / `No issues flagged.` schema in `agents/auditor.md`. Evals pass.
- **Evidence-backed:** no new category, rule, or heuristic without a fixture in `evals/fixtures/` demonstrating it.
- **Graceful on malformed input:** every preprocessing path handles missing session files, empty transcripts, and malformed JSONL without crashing.
- **Lean:** Python stdlib only. No new dependencies without explicit discussion.
- **Honest limitations:** known gaps are listed in `README.md`. Don't ship a change that invalidates that list without updating it.
