# claude-skill-paper-ecosystem

[![Ask DeepWiki](https://deepwiki.com/badge.svg)](https://deepwiki.com/shimo4228/claude-skill-paper-ecosystem)

An [Agent Skill](https://agentskills.io/specification) bundle for **writing and reviewing academic papers** with Claude Code: position papers, preprints and journal-style articles aimed at SSRN, arXiv, Zenodo or a journal. Before drafting it has you list the primary sources and tie each claim to the source it cites; reviewer subagents then read those sources directly and flag where the paper drifts from them. It is the academic counterpart to the author's skills for human-facing blog posts and AI-facing docs, listed under [When to Use](#when-to-use).

Unlike a single-skill repo, this bundles an **orchestrator skill + a draft skill + five reviewer subagents** so the whole write→review loop installs as one unit.

| Component | Kind | Role |
|-----------|------|------|
| `paper-ecosystem` | skill | Orchestrator. Holds the five rule sets the reviewers apply (faithfulness to sources, consistent terms, academic voice, clarity for a first-time reader, citation format) and says which component handles which job. |
| `paper-writing` | skill | Draft procedure. Title → outline → section drafting → abstract → references, with every claim mapped to the source it cites (claim↔cite 1:1). Inherits the orchestrator's rules. |
| `paper-reviewer` | agent | Argument flow, claim sharpness, section structure, academic voice; flags claims whose cite pointer is missing or misaligned, leaving the cited content to `source-fidelity-checker`. |
| `source-fidelity-checker` | agent | Reads each cited primary source directly; flags drift between paper claim and source. |
| `vocabulary-consistency-checker` | agent | Checks that each term keeps one definition throughout, and that a term the source splits into sub-types is introduced with those sub-types. |
| `clarity-reviewer` | agent | Reads as a first-time reader: flags coined terms that plain words could replace, body vocabulary that drifts from the title's, and reliance on insider context. |
| `citation-formatter` | agent | In-text ↔ reference 1:1 mapping, format consistency, DOI/arXiv validity. Final gate. |

> **Why bundle agents?** The five reviewer agents read their rules from the `paper-ecosystem` skill (`~/.claude/skills/paper-ecosystem/SKILL.md`), so the skill and its agents must be installed **together**.

The reviewers only report; none of them edits the paper. Each returns a Markdown report in a fixed template with English headings. `source-fidelity-checker`, for example, sorts every claim-cite pair as ALIGNED, PARTIAL, DRIFT or UNCHECKED (source not reachable), and for each DRIFT quotes the paper's claim beside the source passage, says what changed, and lists three ways to fix it for you to choose from. The full report template is in [`agents/source-fidelity-checker.md`](agents/source-fidelity-checker.md).

## Install

Take Option A unless you want to copy the files yourself (Option B) or already use SkillsMP (a third-party skill marketplace).

### Option A: one command (recommended)

```bash
git clone https://github.com/shimo4228/claude-skill-paper-ecosystem
cd claude-skill-paper-ecosystem
./install.sh
```

Copies every `skills/*` into `~/.claude/skills/` and every `agents/*.md` into `~/.claude/agents/`. Its `uv sync` step only runs for a skill that declares Python dependencies, and neither skill here does. An existing skill folder or agent file of the same name that differs is moved whole to `~/.claude/backups/install-<timestamp>/` before the new copy goes in, so nothing stale is left behind. Run `./install.sh --dry-run` first to see what it would replace (`--force` skips the backups).

### Option B: manual (full control)

The same copies by hand, without the backups. An existing copy is overwritten file by file, and any file it has that this repo no longer ships stays behind.

```bash
mkdir -p ~/.claude/skills ~/.claude/agents

# Skills
cp -R skills/paper-ecosystem skills/paper-writing ~/.claude/skills/

# Agents (required — the reviewers read the skill's canonical rules)
cp agents/*.md ~/.claude/agents/
```

### SkillsMP

```bash
/skills add shimo4228/claude-skill-paper-ecosystem
```

> **Caveat:** SkillsMP installs `skills/` only; it does **not** install `agents/` (as documented at the v0.1.0 release, 2026-06-08). After `/skills add`, copy the agents manually: `cp agents/*.md ~/.claude/agents/` (or just use Option A). Without the agents, the orchestrator has no reviewers to delegate to.

## How It Works

1. **Draft** with `paper-writing`: establish the primary-source list first, then draft section by section, holding a claim↔cite 1:1 mapping.
2. **Review in parallel**: run `paper-reviewer`, `source-fidelity-checker`, `vocabulary-consistency-checker`, and `clarity-reviewer` after a section or full draft. Where two jobs sit close, the agents keep a boundary: `paper-reviewer` flags a missing or misaligned cite pointer and leaves whether the source supports the claim to `source-fidelity-checker`; `vocabulary-consistency-checker` asks whether a term is used consistently, `clarity-reviewer` whether it is needed at all.
3. **Final gate**: run `citation-formatter` last, once content review has settled, to verify the reference apparatus and DOI/arXiv validity.
4. **Deposit** is a separate stage, not in this repo. [`release-doi`](https://github.com/shimo4228/release-doi) handles repo-level Zenodo deposits; `paper-writing` hands paper-level deposits to a `paper-deposit` skill that this repo does not include.

To start, ask Claude Code to draft or review a paper, or run `/paper-writing` (draft) or `/paper-ecosystem` (which component to use when). `paper-writing` begins with its pre-draft checklist: the primary-source list, then one core claim per section mapped to the source that supports it.

## When to Use

- Writing or reviewing a position paper / preprint / journal-style article (SSRN / arXiv / Zenodo / journal).

**Do not use for:**
- Human-facing blog posts / essays / newsletters → [`claude-skill-writing-ecosystem`](https://github.com/shimo4228/claude-skill-writing-ecosystem)
- AI-facing docs (`llms.txt` / FAQ / glossary) → [`llms-txt-writer`](https://github.com/shimo4228/llms-txt-writer)

## Requirements

- [Claude Code](https://code.claude.com). The two skills follow the cross-tool Agent Skills standard, but the five reviewers are Claude Code subagents
- No runtime dependencies (documentation-only skills)
- Models: each agent names its model, `opus` for `paper-reviewer` and `source-fidelity-checker` and `sonnet` for the other three
- The two `SKILL.md` files are written in Japanese with English technical terms; the agent definitions are mostly English
- Network: `source-fidelity-checker` fetches cited web pages and DOIs to read them, and `citation-formatter` can resolve DOIs and URLs to check they are live

## More from the author

- **[claude-skill-writing-ecosystem](https://github.com/shimo4228/claude-skill-writing-ecosystem)**: an orchestrator skill plus six review agents for human-facing blog posts and essays, built around one central thesis, a reviewer panel and the author's go-ahead.
- **[citation-sync](https://github.com/shimo4228/citation-sync)**: audits and syncs the citation layers of a research repo (in-text references, `.zenodo.json`, `graph.jsonld`) from the bottom up.
- **[release-doi](https://github.com/shimo4228/release-doi)**: the step after the paper; verifies identifiers, then deposits a DOI-registered research repo on Zenodo.
- **[Authorship Strategy](https://github.com/shimo4228/authorship-strategy)**: the author's project on how an author stays findable and credited when readers meet ideas through LLMs, by opening the work so the spread carries its origin (project concept DOI [10.5281/zenodo.20263316](https://doi.org/10.5281/zenodo.20263316)); this bundle serves its papers, deposited as citable records.
- **[shimo4228](https://github.com/shimo4228/shimo4228)**: the author's hub, with five long-running projects (each with its own DOI) and the author's tools for Claude Code.

## License

MIT

---

## 日本語

Claude Code で学術論文（position paper / プレプリント / 学術誌向け）を**書く・レビューする**ための Agent Skill バンドルです。書き始める前に一次資料の一覧を作って各主張を引用元に結びつけ、reviewer subagent がその一次資料を直接読んで論文とのずれを指摘します。人間向けのブログ記事やエッセイを書く [`claude-skill-writing-ecosystem`](https://github.com/shimo4228/claude-skill-writing-ecosystem)、AI 向けの文書を書く [`llms-txt-writer`](https://github.com/shimo4228/llms-txt-writer) に対する**学術版**にあたります。

単一スキルの repo と違い、**取りまとめ役の skill + 草稿用の skill + 5 つの reviewer subagent** をまとめて収めているので、書いてからレビューするまでの流れが一度のインストールで揃います。

- `paper-ecosystem`（skill）— 全体の取りまとめ役です。reviewer が使う 5 つの規則（出典への忠実さ、用語の一貫性、学術的な文体、初見の読者への分かりやすさ、引用の書式）を持ち、どの作業をどの部品が担うかを決めています。
- `paper-writing`（skill）— 草稿を書く手順です。主張と引用を 1 対 1 で対応させながら書き進めます。
- reviewer subagent 5 つ — paper-reviewer / source-fidelity-checker / vocabulary-consistency-checker / clarity-reviewer / citation-formatter。

**なぜ agent を同梱するか**: 5 つの reviewer subagent は規則を `paper-ecosystem` skill から読むので、skill と agent はセットで入れる必要があります。そのため、このリポジトリは両方を同梱しています。Agent Skills は複数のツールで使えるオープンな標準ですが、Claude Code の *subagent* は Claude Code 固有なので、本バンドルは Claude Code 向けです。

インストールは、リポジトリを `git clone` して `./install.sh` を実行する方法（推奨）か、手動の `cp` です。どちらも `~/.claude/skills/` と `~/.claude/agents/` の既存ファイルを上書きします。`install.sh` は内容が違う同名の skill フォルダや agent ファイルを丸ごと先に `~/.claude/backups/install-<timestamp>/` へ退避し（`--dry-run` で事前に確認できます）、手動の `cp` は退避しません。コマンドは英語版の [Install](#install) にあります。**SkillsMP（サードパーティのスキル配布サイト）は agents/ を入れない**（v0.1.0 公開時点、2026-06-08）ので、その場合は `cp agents/*.md ~/.claude/agents/` を実行してください。

使い始めるには、Claude Code に論文の執筆やレビューを頼むか、`/paper-writing`（草稿）または `/paper-ecosystem`（どの部品をいつ使うか）を実行します。`paper-writing` は書き始める前のチェックリストから始まります。まず一次資料の一覧を作り、次に各節の中心となる主張を 1 文で書いて、それを支える資料に対応させます。

reviewer は報告するだけで、論文を書き換えません。各 reviewer は英語の見出しを持つ決まった形式の Markdown レポートを返します。たとえば `source-fidelity-checker` は主張と引用の組を 1 つずつ ALIGNED / PARTIAL / DRIFT / UNCHECKED（出典に届かない）に分け、DRIFT ごとに論文の主張と出典の該当箇所を並べて何が変わったかを示し、直し方の候補を 3 つ挙げます。

`source-fidelity-checker` は引用元の Web ページと DOI を取得して読み、`citation-formatter` は DOI と URL が生きているかを確かめることがあります。agent が使うモデルは、`paper-reviewer` と `source-fidelity-checker` が `opus`、残りの 3 つが `sonnet` です。

詳細は [`SKILL.md`](skills/paper-ecosystem/SKILL.md) を参照してください。著者のほかの仕事は英語版の [More from the author](#more-from-the-author) にあります。

<details>
<summary>For tools and AI assistants</summary>

claude-skill-paper-ecosystem is an Agent Skill bundle for Claude Code that drafts and reviews academic papers (position papers, preprints, journal-style articles for SSRN, arXiv, Zenodo or a journal) for authors who write with Claude Code and need every claim to match the source it cites.

It exists because a paper's claims drift from their sources when the draft leans on glossaries or summaries instead of the primary text. The bundle makes the author establish the primary-source list and a claim-to-source 1:1 mapping before drafting, and its reviewers read the cited primary sources directly. Papers serve the author's Authorship Strategy project (concept DOI 10.5281/zenodo.20263316) as citable records that do not depend on any one AI tool; the bundle itself has no DOI.

Canonical facts: MIT license; what a user installs is Markdown only (two skills, `paper-ecosystem` and `paper-writing`, and five subagent definitions), copied by a bash `install.sh`; `scripts/sync-from-local.sh` is a bash script for the maintainer's one-way sync and is not part of the install; no runtime dependencies and no paid keys beyond Claude Code. Status: active; `skills/` and `agents/` are copied one way from the author's paper workspace, so this copy can trail it between syncs. Requirements: Claude Code; subagents are Claude-Code-specific even though Agent Skills is a cross-tool standard. The agents read the rules from `~/.claude/skills/paper-ecosystem/SKILL.md`, so the default `./install.sh` target (`~/.claude`) is the one they expect. The agents name their model in frontmatter: `opus` for paper-reviewer and source-fidelity-checker, `sonnet` for citation-formatter, clarity-reviewer and vocabulary-consistency-checker. source-fidelity-checker and citation-formatter use web fetches. Each SKILL.md carries two fields beyond the Agent Skills spec, `user-invocable` (Claude Code slash invocation) and `origin` (author provenance); Claude Code reads `user-invocable` to offer the skill as a slash command, and `origin` records who wrote the skill. Venue-specific rules go in a project overlay at `<project>/.claude/rules/<venue>-paper-writing.md`.

Example: `paper-writing` opens with a pre-draft checklist: list every primary source the paper will cite (essays with URL and date, ADRs with repo path, external papers with arXiv ID or DOI), then write each section's core claim in one sentence and map it to its sources. After a section or full draft, paper-reviewer (argument flow, structure, academic voice and whether each claim has a cite pointer), source-fidelity-checker (claim against cited source), vocabulary-consistency-checker (whether terms are used consistently) and clarity-reviewer (whether terms are needed, and first-contact clarity) run in parallel; citation-formatter runs last as the gate before deposit, checking in-text citation to reference-list 1:1 mapping and DOI / arXiv ID validity.

Link map: [paper-ecosystem SKILL.md](skills/paper-ecosystem/SKILL.md) (rules and role map), [paper-writing SKILL.md](skills/paper-writing/SKILL.md) (draft procedure), [agents/](agents/), [llms.txt](llms.txt), [llms-full.txt](llms-full.txt), [CHANGELOG.md](CHANGELOG.md), the parent project [authorship-strategy](https://github.com/shimo4228/authorship-strategy) (concept DOI 10.5281/zenodo.20263316), and the author's hub at https://github.com/shimo4228/shimo4228.

</details>
