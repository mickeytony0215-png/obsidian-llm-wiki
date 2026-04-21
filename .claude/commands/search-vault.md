---
name: search-vault
description: Structured and semantic search over the vault — frontmatter queries, content grep, backlink graph, orphan detection, and optional semantic search via qmd
---

A read-only query layer for the vault. Six modes covering frontmatter filters, full-text grep, link-graph analysis, and optional semantic search. Never modifies files.

> **This repo ships the command spec only.** The Python backend is intentionally not bundled. Fork users either reimplement `.claude/scripts/search_vault.py` following this spec, or replace the six modes with their preferred tooling (Obsidian Dataview queries, shell scripts, etc.). The command file below describes the intended interface and behaviour.

## Use cases

- Find papers matching a metadata filter (e.g. `reviewed_with_notes`, 2024, a specific tag).
- Find inbound references to a given page (who cites this concept or paper?).
- Find orphan pages with zero inbound wikilinks — complements `/lint-wiki`.
- Full-text search (ripgrep) with structured markdown output.
- Vault statistics: review-status distribution, year histogram, tag frequency.
- Semantic search when the exact wording is unknown — finds concept-level matches via BM25 + vector + LLM rerank (requires qmd).

## Architecture

```
/search-vault
      |
      v
  .claude/scripts/search_vault.py         <- fork users implement this
      |
      +- structured   -> frontmatter parser (stdlib only)
      +- content      -> ripgrep (fallback: Python regex)
      +- orphan       -> scan all [[wikilinks]], build link graph
      +- inbound      -> backlinks for a target page
      +- stats        -> frontmatter aggregations
      +- semantic     -> qmd CLI (optional external dependency)
```

## Usage

### 1. Structured query (frontmatter filters)

```bash
python .claude/scripts/search_vault.py structured --status reviewed_with_notes
python .claude/scripts/search_vault.py structured --tag <domain-tag> --year 2024
python .claude/scripts/search_vault.py structured --type ppt-draft
```

Common flag combinations:

- `--status disputed` — list every paper flagged as disputed
- `--status unreviewed` — the pending-review queue
- `--tag <concept>` — same-topic papers
- `--year 2025` — this year's papers

### 2. Content search (full-text grep)

```bash
python .claude/scripts/search_vault.py content "<substring or regex>"
```

Output is markdown with `[[wikilink]]` results, ready to paste into a note.

### 3. Orphan detection

```bash
python .claude/scripts/search_vault.py orphan
```

Lists pages with zero inbound `[[wikilinks]]`, excluding meta pages (log, index, dashboard, pending-review, review-history, SKILL).

### 4. Inbound search (who cites this page?)

```bash
python .claude/scripts/search_vault.py inbound "<target page name>"
```

Supports these wikilink forms:

- `[[target]]`
- `[[target|alias]]`
- `[[target#section]]`
- `[[path/to/target]]`

### 5. Stats (vault summary)

```bash
python .claude/scripts/search_vault.py stats
```

Output: total file count, review-status distribution, year distribution, top-20 tags.

### 6. Semantic search (requires qmd)

```bash
# Hybrid (BM25 + vector + LLM rerank, recommended)
python .claude/scripts/search_vault.py semantic "<query in natural language>"

# Vector only (pure semantic)
python .claude/scripts/search_vault.py semantic "<query>" --mode vsearch

# BM25 only (pure keyword, fast)
python .claude/scripts/search_vault.py semantic "<query>" --mode search
```

Three mode choices:

| Mode | Purpose | Latency |
|---|---|:---:|
| `query` (default) | Most precise; hybrid with LLM rerank | Slow (~3-10s) |
| `vsearch` | Pure semantic; finds concept-similar text | Medium (~1-3s) |
| `search` | Pure keyword; exact-string matches | Fast (<1s) |

When to use `semantic` vs `content`:

- Use `content` when you know the exact substring and want every occurrence.
- Use `semantic query` when you don't remember the wording and want meaning-similar paragraphs.
- Use `semantic vsearch` to find papers with a similar design philosophy to a known one.
- Use `semantic search` for cross-language or cross-vocabulary lookups (e.g. searching "TA" also matches "trusted authority").

First-run cost when using qmd:

- Downloads embedding model (~328 MB)
- Downloads reranker model (~600 MB)
- Downloads query-expansion model (~1.28 GB)
- Initial vault embedding (~4 minutes per ~200 notes)
- Cached permanently after that; subsequent queries typically < 5 seconds.

See the [qmd project](https://github.com/tobi/qmd) for installation details.

## Typical flows

### Preparing a lab-seminar talk

```bash
# 1. Find inbound references to the focal paper
python .claude/scripts/search_vault.py inbound "<paper name>"

# 2. Find other papers on the same topic
python .claude/scripts/search_vault.py structured --tag <topic>

# 3. Check which same-topic papers are reviewed_with_notes (so citations carry the right caveat)
python .claude/scripts/search_vault.py structured --status reviewed_with_notes
```

### Writing a synthesis page (cross-paper pattern-finding)

```bash
# 1. Find every mention of a term
python .claude/scripts/search_vault.py content "<term>"

# 2. Find papers tagged for that theme
python .claude/scripts/search_vault.py structured --tag <theme>
```

### Periodic vault health check

```bash
python .claude/scripts/search_vault.py stats
python .claude/scripts/search_vault.py orphan
```

## Output format

Every mode produces a markdown report, drop-in for notes.

Example (structured):

```markdown
# Structured Search Result

**Filters**: status=reviewed_with_notes
**Matches**: 9

- [[paper-name-1]] -- status: `reviewed_with_notes`, year: 2024
- [[paper-name-2]] -- status: `reviewed_with_notes`, year: 2025
- ...
```

## How it complements other commands

| Tool | Purpose | Relationship |
|---|---|---|
| `/search-vault` | Structured queries, backlinks, orphan detection, content grep | new read-only query tool |
| `/lint-wiki` | Wiki structural + cascade-caveat audit | parallel (lint audits health; search finds things) |
| `/lint-related-work` | Related Work framing fairness check | parallel (lint targets Related Work sections) |
| `/organize-vault` | Full-vault re-linking | parallel (organize mutates; search is read-only) |
| Obsidian built-in search | Interactive full-text search in the GUI | complementary (GUI for exploration; CLI for batch and for pasting results into notes) |

## Limits

- Requires Python 3.10+ (3.13 verified against).
- No fuzzy matching (strict string or regex).
- No pure-Python semantic similarity; semantic mode needs qmd.
- Frontmatter parser is simplified (no deep nesting, no multi-line strings). Sufficient for typical vaults.
- Inbound search is regex-based; imperfect but coverage is sufficient in practice.

## Future extensions

- `--pdf-link` mode: find notes referencing `[[raw/papers/xxx.pdf]]` and check that the PDF exists.
- `--broken-links` mode: find wikilinks pointing to non-existent files.
- `--similar <note>` mode (requires qmd): find notes semantically similar to a target.

## References

- Design inspiration: Karpathy's [LLM Wiki gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
- Semantic backend: [qmd by tobi](https://github.com/tobi/qmd)

## Notes

- This command is read-only; it never modifies any file.
- Large vaults may need a few seconds for `orphan` mode (full vault scan).
- Semantic mode needs qmd installed separately; all other modes are pure Python.
