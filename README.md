# claude-skill-paper-ecosystem

[![Ask DeepWiki](https://deepwiki.com/badge.svg)](https://deepwiki.com/shimo4228/claude-skill-paper-ecosystem) [![GitMCP](https://img.shields.io/endpoint?url=https://gitmcp.io/badge/shimo4228/claude-skill-paper-ecosystem)](https://gitmcp.io/shimo4228/claude-skill-paper-ecosystem) [![View Code Wiki](https://assets.codewiki.google/readme-badge/static.svg)](https://codewiki.google/github.com/shimo4228/claude-skill-paper-ecosystem)

An [Agent Skill](https://agentskills.io/specification) bundle for **writing and reviewing academic papers** — position papers, preprints, and journal-style articles aimed at SSRN / arXiv / Zenodo / journal venues. It is the academic counterpart to [`claude-skill-writing-ecosystem`](https://github.com/shimo4228/claude-skill-writing-ecosystem) (human-facing blog/essay) and [`llms-txt-writer`](https://github.com/shimo4228/llms-txt-writer) (AI-facing docs).

Unlike a single-skill repo, this bundles an **orchestrator skill + a draft skill + five reviewer subagents** so the whole write→review loop installs as one unit.

| Component | Kind | Role |
|-----------|------|------|
| `paper-ecosystem` | skill | Orchestrator. Holds the canonical rules (Source Fidelity, Vocabulary Consistency, Academic Voice, Reader Clarity, Citation Format) and the role-boundary map. |
| `paper-writing` | skill | Draft procedure. Title → outline → section drafting → abstract → references, with claim↔cite 1:1 mapping enforced. Inherits the orchestrator's rules. |
| `paper-reviewer` | agent | Argument flow, claim sharpness, evidence-claim alignment, section structure. |
| `source-fidelity-checker` | agent | Reads each cited primary source directly; flags drift between paper claim and source. |
| `vocabulary-consistency-checker` | agent | Term-definition consistency and sub-classification discipline. |
| `clarity-reviewer` | agent | First-contact reader clarity (coined-term budget, title-axis carry-through, no insider context). |
| `citation-formatter` | agent | In-text ↔ reference 1:1 mapping, format consistency, DOI/arXiv validity. Final gate. |

> **Why bundle agents?** The five reviewer agents read their canonical rules from the `paper-ecosystem` skill (`~/.claude/skills/paper-ecosystem/SKILL.md`). The skill and its agents must be installed **together** — that is what this repo packages. Agent Skills are an open cross-tool standard, but Claude Code *subagents* are Claude-Code-specific, so this bundle targets Claude Code.

## Install

This repo bundles **both skills and the agents they depend on**.

### Option A — one command (recommended)

```bash
git clone https://github.com/shimo4228/claude-skill-paper-ecosystem
cd claude-skill-paper-ecosystem
./install.sh
```

Copies every `skills/*` into `~/.claude/skills/`, every `agents/*.md` into `~/.claude/agents/`, and runs `uv sync` for any skill that declares Python dependencies. Existing files are backed up to `*.bak-<timestamp>` before being replaced (use `--force` to skip backups, `--dry-run` to preview).

### Option B — manual (full control)

```bash
# Skills
cp -r skills/paper-ecosystem ~/.claude/skills/paper-ecosystem
cp -r skills/paper-writing   ~/.claude/skills/paper-writing

# Agents (required — the reviewers read the skill's canonical rules)
cp agents/*.md ~/.claude/agents/
```

These skills are documentation-only; no `uv sync` step is required.

### SkillsMP

```bash
/skills add shimo4228/claude-skill-paper-ecosystem
```

> **Caveat:** SkillsMP installs `skills/` only — it does **not** install `agents/`. After `/skills add`, copy the agents manually: `cp agents/*.md ~/.claude/agents/` (or just use Option A). Without the agents, the orchestrator has no reviewers to delegate to.

## How It Works

1. **Draft** with `paper-writing` — establish the primary-source list first, then draft section by section, holding a claim↔cite 1:1 mapping.
2. **Review in parallel** — run `paper-reviewer`, `source-fidelity-checker`, `vocabulary-consistency-checker`, and `clarity-reviewer` as orthogonal lenses after a section or full draft.
3. **Final gate** — run `citation-formatter` last, once content review has settled, to verify the reference apparatus and DOI/arXiv validity.
4. **Deposit** is a separate stage (not in this repo) — see `release-doi` for repo-level Zenodo deposits; a `paper-deposit` skill covers paper-level deposits.

## When to Use

- Writing or reviewing a position paper / preprint / journal-style article (SSRN / arXiv / Zenodo / journal).

**Do not use for:**
- Human-facing blog posts / essays / newsletters → [`claude-skill-writing-ecosystem`](https://github.com/shimo4228/claude-skill-writing-ecosystem)
- AI-facing docs (`llms.txt` / FAQ / glossary) → [`llms-txt-writer`](https://github.com/shimo4228/llms-txt-writer)

## Requirements

- [Claude Code](https://code.claude.com) (or another agent product that supports the Agent Skills standard and subagents)
- No runtime dependencies (documentation-only skills)

## Related skills (siblings)

- [`claude-skill-writing-ecosystem`](https://github.com/shimo4228/claude-skill-writing-ecosystem) — human-facing writing & review (also bundles its agents)
- [`llms-txt-writer`](https://github.com/shimo4228/llms-txt-writer) — AI-facing documents
- [`readme-writer`](https://github.com/shimo4228/readme-writer) — human-facing READMEs

## About this skill

This bundle is a **component of the [Authorship Strategy](https://github.com/shimo4228/authorship-strategy) research line** ([DOI 10.5281/zenodo.20263316](https://doi.org/10.5281/zenodo.20263316)) maintained by [@shimo4228](https://github.com/shimo4228). It operationalizes the academic-publishing surface of the program's diffusion tactics: harness-neutral papers deposited as citable records.

Frontmatter note: each `SKILL.md` carries two harness extensions beyond the [Agent Skills spec](https://agentskills.io/specification) — `user-invocable` (Claude Code slash-invocation) and `origin` (author provenance tracking). Both are tolerated as extra fields by the standard.

## License

MIT

---

## 日本語

学術論文（position paper / preprint / journal 向け）を**書く・レビューする**ための Agent Skill バンドルです。人間向け blog/essay の [`writing-ecosystem`](https://github.com/shimo4228/claude-skill-writing-ecosystem)、AI 向け doc の [`llms-txt-writer`](https://github.com/shimo4228/llms-txt-writer) に対する**学術版**にあたります。

単一スキルの repo と違い、**orchestrator skill + draft skill + 5 つの reviewer subagent** を一括同梱し、write→review ループ全体が 1 単位で入ります。

- `paper-ecosystem`（skill）— orchestrator。canonical rules（Source Fidelity / Vocabulary Consistency / Academic Voice / Reader Clarity / Citation Format）と役割境界の正本。
- `paper-writing`（skill）— draft 手順。claim↔cite の 1:1 mapping を強制。
- reviewer agents 5 種 — paper-reviewer / source-fidelity-checker / vocabulary-consistency-checker / clarity-reviewer / citation-formatter。

**なぜ agent を同梱するか**: 5 つの reviewer agent は canonical rules を `paper-ecosystem` skill から読みます。skill と agent はセットで入れる必要があり、それを本 repo が梱包します。Agent Skills はオープンな cross-tool 標準ですが、Claude Code の *subagent* は Claude Code 固有なので、本バンドルは Claude Code 向けです。

インストールは `./install.sh`（推奨）か手動 `cp`。**SkillsMP は agents/ を入れない**ので、その場合は `cp agents/*.md ~/.claude/agents/` を実行してください。

詳細は各 [`SKILL.md`](skills/paper-ecosystem/SKILL.md) を参照してください。
