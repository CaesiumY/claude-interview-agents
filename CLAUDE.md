# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Claude Code **plugin** for frontend developers (3 years experience) preparing for job interviews.
Covers the full journey: resume review → technical interview prep → executive/culture-fit mock
interviews → portfolio building (4-Phase harness based on
https://caesiumy.dev/posts/ai/portfolio-strategy-harness-design/).

## Plugin Structure

This repo IS a plugin (root-level layout, not `.claude/`):

- `.claude-plugin/plugin.json` — plugin manifest (name: `interview-agents`)
- `.claude-plugin/marketplace.json` — self-hosted marketplace (install via `claude plugin marketplace add caesiumy/claude-interview-agents`)
- `commands/` — 5 user-invocable commands (flat .md files)
- `agents/` — 6 subagents (all pinned to `model: claude-opus-4-8`, see Model Swap below)
- `skills/<name>/SKILL.md` — reference guides loaded by commands via `${CLAUDE_PLUGIN_ROOT}`
- `portfolio/`, `resumes/` — output conventions only; actual outputs go to the **user's cwd**

When installed, commands are namespaced: `/interview-agents:review-resume` etc.
Local dev: `claude --plugin-dir .` and `claude plugin validate .`

## Commands

| Command | Purpose | Output (cwd-relative) |
|---------|---------|----------------------|
| `/review-resume <이력서>` | Score resume /100 with PSR decomposition | `resumes/[name]_review.md` |
| `/create-questionnaire <이력서>` | Generate questionnaire (4-category questions) | `resumes/[name]_questionnaire.md` |
| `/evaluate <질문지>` | 3-agent parallel evaluation | `resumes/[name]_evaluation.md` |
| `/mock-interview <이력서> [--type executive\|culture]` | Interactive mock interview + evaluation | `resumes/[name]_mock_[type]_[date](_evaluation).md` |
| `/portfolio <이력서> [JD]` | 4-Phase portfolio harness | `portfolio/<slug>/...` |

## Agent Architecture

### Technical track (`/evaluate`)
- `technical-evaluator` (strict, 45%) ∥ `communication-evaluator` (positive, 30%) — run in
  **parallel** to prevent anchoring bias → `final-arbiter` (neutral, weights 45/30/25) synthesizes.
- If a mock-interview evaluation file exists, final-arbiter uses it as **reference only**
  (weights unchanged).

### Culture track (`/mock-interview`)
- `culture-fit-evaluator` scores sessions: authenticity 25 / org-fit 25 / growth narrative 20 /
  attitude 15 / risk signals 15.

### Portfolio harness (`/portfolio`) — writer ≠ reviewer separation
- `portfolio-strategist` (Phase 3) reads ONLY `skills/portfolio-strategy/templates.md`
- `portfolio-reviewer` (Phase 4) reads ONLY `skills/portfolio-strategy/review-rubric.md`
- Never merge these two into one agent or one context — this is the core harness design.
- Review thresholds: structure=binary, fit≥70, readability≥75 (word ratio 1:8–1:15), overall≥80.
  Auto-rewrite loop max 3 iterations.
- 3 user gates: GATE 1 (item selection), GATE 2 (interview notes approval), GATE 3·4 (final save).
  Phase 3→4 runs WITHOUT a gate.
- Interview answers are appended to `portfolio/<slug>/interview-notes.md` immediately — they are
  permanent, reusable assets. Never delete or overwrite them.

## Model Swap

All 6 agent files pin `model: claude-opus-4-8` with a `# MODEL SWAP POINT` comment directly above.
These 6 frontmatter lines are the ONLY swap points — commands/skills intentionally have no model field.
(History: v2.0.0 shipped on `claude-fable-5`; swapped to Opus on 2026-07-10 ahead of Fable retirement.)

To swap models, run from repo root:

```bash
sed -i 's/<old-model-id>/<new-model-id>/' agents/*.md
```

Then verify: `grep -r "<old-model-id>" agents/` must return nothing.

## Conventions

- All content in Korean (target user is a Korean frontend developer)
- Output files written UTF-8, paths relative to the user's cwd (plugin installs to cache dir)
- Agent JSON output schemas follow the pattern in `agents/technical-evaluator.md`
- Skill references in commands must use `${CLAUDE_PLUGIN_ROOT}/skills/<name>/...` — never
  relative paths (they break after plugin installation)

## Judgment Criteria (final-arbiter)

- 85-100: Strong Pass / 70-84: Pass / 60-69: Hold / 50-59: Conditional / 0-49: Fail
