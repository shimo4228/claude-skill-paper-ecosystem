# Changelog

All notable changes to this project will be documented in this file.

## [0.1.0] — 2026-06-08

Initial public release.

### What it does

An [Agent Skill](https://agentskills.io/specification) bundle for writing and
reviewing academic papers (position papers, preprints, journal-style articles
for SSRN / arXiv / Zenodo / journal venues). The academic counterpart to
[claude-skill-writing-ecosystem](https://github.com/shimo4228/claude-skill-writing-ecosystem)
(human-facing) and
[claude-skill-llms-txt-writer](https://github.com/shimo4228/claude-skill-llms-txt-writer)
(AI-facing).

### First repo to bundle skills + agents together

This is the first ecosystem repo to ship an **orchestrator skill + draft skill +
five reviewer subagents** as one installable unit, via `install.sh`. The five
reviewer agents read their canonical rules from the `paper-ecosystem` skill, so
they must be installed together.

### Components

Skills:
- `paper-ecosystem` — orchestrator; canonical rules (Source Fidelity,
  Vocabulary Consistency, Academic Voice, Reader Clarity, Citation Format) and
  role-boundary map.
- `paper-writing` — draft procedure (title → outline → sections → abstract →
  references) with claim↔cite 1:1 mapping enforced.

Agents (`agents/`):
- `paper-reviewer` — argument flow, claim sharpness, evidence-claim alignment.
- `source-fidelity-checker` — reads cited primary sources directly; flags drift.
- `vocabulary-consistency-checker` — term-definition consistency.
- `clarity-reviewer` — first-contact reader clarity.
- `citation-formatter` — citation ↔ reference mapping, DOI/arXiv validity (final gate).

### Convention

- `install.sh` (idempotent, backup-then-overwrite, `--force` / `--dry-run`) copies
  `skills/*` → `~/.claude/skills/` and `agents/*.md` → `~/.claude/agents/`.
- SkillsMP installs `skills/` only; agents must be copied manually (documented in README).

### Scope note

Paper deposit (Zenodo / SSRN) is a separate stage and is **not** in this repo —
see `release-doi` for repo-level deposits; a `paper-deposit` skill covers
paper-level deposits.

### Requirements

- Claude Code (or another agent product supporting the Agent Skills standard + subagents)
- No runtime dependencies (documentation-only skills)
