# obsidian-llm-wiki

**Turn a growing pile of academic papers into a self-maintaining, citable research wiki — with [Claude Code](https://claude.com/claude-code) and [Obsidian](https://obsidian.md).**

Twelve slash commands wire a three-layer architecture (raw sources → reviewed notes → compiled wiki) under one rule: **no auto-generated claim enters the wiki until a human verifies it against the original PDF.** No cloud service beyond Claude. Your papers, notes, and knowledge graph all live on your disk.

This repo ships **only the workflow** — the slash commands, the schema, the Obsidian config. It does not ship the author's personal research notes, PDFs, or filled-in profile. Clone it, drop in your own PDFs, edit `SKILL.md` once, and you have a reproducible paper-to-wiki pipeline. The design follows Andrej Karpathy's [LLM wiki pattern](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) (three-layer architecture: raw / wiki / schema; three operations: ingest / query / lint), extended with a **review-gatekeeping layer** that prevents unreviewed notes from polluting the wiki, a **paper-to-presentation pipeline** that turns reviewed notes into seminar drafts and slide material, and a **read-only query layer** (`/search-vault`) with structured filters, backlink / orphan graphs, and optional semantic search.

## Is this for you?

**Pick this up if you…**

- Read two or more academic papers a week and want them to compound into something you can query, not just a file cabinet.
- Already use, or are willing to adopt, Claude Code and Obsidian.
- Have been burned by LLMs confidently "summarising" a paper with details they invented, and want tooling that catches that before it spreads through your notes.
- Prefer a local-first setup you can edit and fork over a hosted SaaS.

**Skip this if you…**

- Want a plug-and-play note-taking app. This is an opinionated workflow; expect to edit a SKILL file and a few templates.
- Want an LLM to auto-summarise hundreds of PDFs without a human review step. The review gate is mandatory by design.
- Are primarily looking for a citation / reference manager. Zotero and friends remain better at that; this repo complements them rather than replacing them.

## A 30-second tour

A typical paper flowing through the pipeline:

```
> /read-paper paper.pdf        # reads the PDF, produces a structured note marked `unreviewed`
> /review-note paper           # claim-by-claim audit against the PDF; sets `reviewed` / `reviewed_with_notes` / `disputed`
> /lint-related-work paper     # (optional) Related Work framing fairness check
> /ingest-to-wiki paper        # hard-refuses if still unreviewed; builds the wiki source + concept pages
> /paper-report paper          # (optional) seminar-report draft
> /paper-ppt    paper          # (optional) bilingual slide material (slides in English, speaker notes in your language)
> /lint-wiki                   # every 5–10 papers: orphan pages, broken links, cascade-caveat audit
> /search-vault <mode>         # read-only queries: structured / content / orphan / inbound / stats / semantic
```

What you get after a handful of papers: a `wiki/` folder with source pages per paper, concept pages with cross-paper comparisons, a synthesis layer for literature reviews, and a Dataview-powered index that keeps itself up to date. Every citation carries a cascade caveat reflecting its paper's review status — a `disputed` paper cannot quietly be cited as fact three pages deep.

## Contents

1. [Is this for you?](#is-this-for-you)
2. [A 30-second tour](#a-30-second-tour)
3. [The three layers](#the-three-layers)
4. [The twelve slash commands](#the-twelve-slash-commands)
5. [Typical workflow](#typical-workflow)
6. [The review gate](#the-review-gate)
7. [Setup](#setup)
8. [What is and isn't in this repo](#what-is-and-isnt-in-this-repo)
9. [Non-goals](#non-goals)
10. [Credit and license](#credit-and-license)

## The three layers

```
raw/                      # Immutable sources (PDFs, clippings) — gitignored
wiki/                     # LLM-compiled knowledge base
  ├── index.md            # Dataview-generated master index
  ├── dashboard.md        # Health-check queries
  ├── concepts/           # One page per reusable concept
  ├── entities/           # Tools, standards bodies, algorithms, teams
  ├── sources/            # One page per paper, with structured metadata
  └── synthesis/          # Cross-source comparative analyses
.claude/
  ├── skills/SKILL.md     # Shared context: user background, writing rules, schema
  └── commands/           # The twelve slash commands documented below
note/Templates/           # Obsidian note templates for new papers
.obsidian/                # Vault config (Dataview, Claudian, BRAT)
```

## The twelve slash commands

Each command is defined as a markdown file in `.claude/commands/`. They're grouped by role in the pipeline.

### Paper intake

| Command | Purpose | Trigger | Output |
|---|---|---|---|
| **`/read-paper`** | Read a PDF and auto-generate a structured paper note. Always sets `review_status: unreviewed` — the note must pass `/review-note` before the wiki will touch it. | After dropping a new PDF into `raw/papers/` | New `.md` note at vault root + forward/back links to existing concept notes + `unreviewed` markers on all touched pages |
| **`/organize-note`** | Rewrite an existing hand-written paper note using the PDF as ground truth. Preserves all personal comments (wrapped in blockquotes). | When you already have rough notes and want a structured version | Overwrites the current note + auto-links + concept extraction |
| **`/organize-vault`** | Scan the entire vault, add missing YAML frontmatter, build bidirectional `[[wikilinks]]`, extract recurring concepts into atomic notes. | Periodically as the vault grows | Modified notes + new concept pages + a report |

### Review

| Command | Purpose | Trigger | Output |
|---|---|---|---|
| **`/review-note`** | The most important command. Full audit of a note against the PDF: claim-by-claim comparison, logical consistency (motivation → method → results → conclusion), and data-credibility red flags (arithmetic-progression tables, "every metric wins" patterns, in-text/table contradictions). Updates `review_status` and appends to `wiki/review-history.md`. | After `/read-paper`, or anytime a note is suspected | Updated frontmatter (`reviewed` / `reviewed_with_notes` / `disputed`), a `## Review log` section in the note, and an audit entry in `wiki/review-history.md` |
| **`/lint-related-work`** | Fairness audit of a paper's Related Work framing. Phase 1 (mechanical): grep for absolutist wording (`completely overlooked`, `entirely fails`, …), extract the baseline list, flag scope mismatches (an authentication paper citing a task-scheduling paper as a baseline, etc.). Phase 2 (optional, needs the baseline PDF in `raw/papers/`): cross-validate each criticised baseline against its own Abstract / Contributions to catch self-contradictory criticisms. Suggests but does not auto-apply `review_status` downgrades. | After `/read-paper`, before `/ingest-to-wiki`, or when preparing a seminar talk on a focal paper | Structured markdown lint report; when issues are found, an entry is appended to `wiki/log.md` |

### Query and discovery

| Command | Purpose | Trigger | Output |
|---|---|---|---|
| **`/search-vault`** | Read-only structured and semantic search over the vault, backed by `.claude/scripts/search_vault.py`. Six modes: `structured` (frontmatter filters — status / tag / year / type), `content` (ripgrep full-text), `orphan` (pages with zero inbound `[[wikilinks]]`, complements `/lint-wiki`), `inbound` (backlinks of a given target, supports `[[alias]]` / `[[#section]]` / `[[path/to/target]]`), `stats` (review-status / year / top-tag distributions), and `semantic` (BM25 + vector + LLM-rerank via [qmd](https://github.com/tobi/qmd), with `query` / `vsearch` / `search` sub-modes for hybrid, pure-vector, or pure-keyword). Never modifies files. | Finding related papers by tag / year / status, checking backlinks before touching a page, auditing orphans, or looking up concepts by meaning rather than exact string | Markdown report with `[[wikilink]]` results — drop-in for notes |

### Note-to-wiki

| Command | Purpose | Trigger | Output |
|---|---|---|---|
| **`/concept-extract`** | Pull 3–5 atomic concepts out of the current note and create or update concept pages. | While reading a note | New concept pages + `[[wikilinks]]` added to the source note |
| **`/ingest-to-wiki`** | **Gatekept**: refuses to run on `unreviewed` notes. Creates a `wiki/sources/` page, updates related concept/entity pages, appends a log entry. | After the note is reviewed | New source page, updated concept pages, `wiki/log.md` entry |
| **`/lint-wiki`** | Health check: orphan pages, broken links, frontmatter completeness (needed for Dataview), and cascade-caveat consistency (every disputed/unreviewed citation must carry its warning). | After ingesting 5–10 new sources | Structured lint report + auto-fixes for mechanical issues |

### Output generation

| Command | Purpose | Trigger | Output |
|---|---|---|---|
| **`/paper-summary`** | Fill in six standard sections (research question / system model / method / key details / pros & cons / relation to my research) on the current paper note. Does not modify `review_status`. | Fleshing out a sparse note | Edited note |
| **`/paper-report`** | Generate a full seminar-report draft from a paper note + PDF, in the preferred output language set in `SKILL.md`. Includes a critical-analysis section that checks motivation/methodology consistency. | Preparing for a lab seminar | New `seminar-report-{paper}-{date}.md` at vault root |
| **`/paper-ppt`** | Generate bilingual slide material: English bullets (≤15 words) on each slide, speaker notes in the preferred output language from `SKILL.md`. 14–18 slides covering Title → Outline → Background → Gap → Related Work → System Model → Method → Experiment → Results → Pros & Cons → Relation to Research → Conclusion → Q&A. Disputed papers get a mandatory warning slide and a "borrow the idea, not the numbers" closer. | Building seminar slides | New `slide-draft-{paper}-{date}.md` |

## Typical workflow

The commands chain together. A typical paper-to-wiki cycle:

```
   raw/papers/foo.pdf
         │
         ▼
   /read-paper foo.pdf          ← creates unreviewed note at vault root
         │
         ▼
   /review-note foo             ← REQUIRED gate: updates review_status
         │
    ┌────┴────┐
    ▼         ▼
  reviewed  disputed
    │         │
    ▼         ▼
   /ingest-to-wiki foo          ← creates wiki/sources/foo, updates concepts
         │
         ▼
   (optional) /paper-report foo    → seminar draft
   (optional) /paper-ppt foo       → slide material
         │
         ▼
   every 5–10 papers: /lint-wiki  ← structural + cascade-caveat audit
```

`/search-vault` sits outside this pipeline — it's a read-only inspection tool usable at any step (find related papers before writing, check backlinks before renaming, audit orphans alongside `/lint-wiki`, or reach for semantic search when you don't remember the exact string).

Three points to internalize:

- **`/ingest-to-wiki` hard-refuses `unreviewed` notes.** This is the core quality gate — see below.
- **`/review-note` is full-depth.** It's not a skim: claim-by-claim PDF comparison, logical consistency, and data red flags. Expect it to take 10–30 minutes per paper.
- **Output commands (summary / report / ppt) propagate the review status.** A `disputed` paper generates a PPT that opens with a warning and ends with "borrow the idea, not the numbers." You cannot accidentally laundering bad data through the pipeline.

## The review gate

This is the main deviation from Karpathy's original pattern. Karpathy's gist treats the LLM's synthesis as authoritative; this repo requires human validation before a note joins the citable knowledge graph.

Four states live in the note's frontmatter:

| State | Meaning | Can wiki cite it? |
|---|---|---|
| `unreviewed` | Auto-generated, not yet verified | No — `/ingest-to-wiki` refuses |
| `reviewed` | User has verified every claim against the PDF | Yes, freely |
| `reviewed_with_notes` | Verified, but minor caveats recorded in `review_notes` | Yes, with caveat in citation |
| `disputed` | Major issues found (fabricated-looking data, method/experiment mismatch, etc.) | Only as "borrow the idea, not the numbers" |

Every wiki page that cites a non-`reviewed` source must carry a **cascade caveat** — a warning marker next to the citation explaining why. `/lint-wiki` audits this automatically. The reasoning: a review system only has value if its results propagate; otherwise a disputed paper ends up cited as fact three pages deep and the wiki becomes a laundromat for bad claims.

Red flags the reviewer actively hunts for (each has burned a real paper):

- Perfect arithmetic progressions in results tables (`9.47 → 7.11 → 4.74 → 2.37` with differences all 2.36±0.01)
- Fixed offset between metrics across all models (Accuracy − Precision is always exactly 0.03)
- Proposed method wins on every single metric (real methods have trade-offs)
- In-text narrative contradicts the table numbers
- Claims high-speed scenario but only simulates low-speed

## Setup

1. **Clone this repo.**
2. **Open the folder as an Obsidian vault.** Obsidian will prompt to install the community plugins listed in `.obsidian/community-plugins.json`: `dataview` (required — `wiki/index.md` queries rely on it), `obsidian42-brat` (for installing Claudian), `claudian` (optional, Claude-in-Obsidian plugin).
3. **Install Claude Code** and place your PDFs under `raw/papers/` (gitignored).
4. **The twelve slash commands are already wired up** — just type `/read-paper` inside Claude Code while the vault is the working directory. `SKILL.md` is loaded as shared context automatically.
5. **Edit `SKILL.md`** at `.claude/skills/SKILL.md`. The top section ("Researcher profile") has placeholders for your role, research domain, output language, and common obligations — fill them in so Claude tailors analysis and output accordingly. Everything below the horizontal rule is the universal workflow spec and should not need editing.

### Recommended customization points

- **`SKILL.md`** — research domain, output language, preferred tools. This is the single biggest lever.
- **`note/Templates/paper-template.md`** — the frontmatter schema for new paper notes.
- **`wiki/index.md`** — Dataview queries. Edit if you rename or add wiki subfolders.
- **Command files under `.claude/commands/`** — tweak trigger phrasing, output format, or review-gate strictness.
- **Your Obsidian look and feel (personal taste, not shared)** — theme, font size, light/dark mode (`.obsidian/appearance.json`) and graph-view layout (`.obsidian/graph.json`) are not tracked in this repo. Set them however you like in `Settings → Appearance` and the graph view; they live only on your machine.

## What is and isn't in this repo

**Included**: all twelve slash commands (the two newest — `/search-vault` and `/lint-related-work` — are shipped as spec-only markdown; their Python implementations are intentionally not bundled, see below), `SKILL.md`, the wiki schema (`index.md`, `dashboard.md`), Obsidian config (app settings and plugin list), Obsidian note templates, and this README.

About `.obsidian/`: only the workflow-relevant config is tracked — `app.json` (contains the paste-image convention `attachmentFolderPath: "./${filename}"`) and `community-plugins.json` (tells Obsidian which plugins to prompt you to install — Dataview, Claudian, BRAT). Personal preferences (theme, graph view, built-in plugin toggles, workspace layout) are all gitignored so they do not overwrite your own settings when you clone this.

About `wiki/dashboard.md` and `wiki/index.md`: both are pre-built Dataview templates. They will look empty when you first open the freshly-cloned vault, because the queries have no pages to aggregate yet. Once you start ingesting papers with `/ingest-to-wiki`, they populate automatically — there is no manual step to "activate" them.

`note/Templates/` holds two Obsidian note templates (paper and seminar-report). They are starting points, not prescriptions: edit the frontmatter fields, section headings, and prompts to match your own reading style.

Note on the spec-only commands:

- `/search-vault` describes a six-mode Python script at `.claude/scripts/search_vault.py` (not shipped). Fork users reimplement it from the spec, or replace it with Dataview queries and shell snippets. Its semantic mode depends on [qmd](https://github.com/tobi/qmd), which is also not bundled; the other five modes (structured / content / orphan / inbound / stats) are pure Python with no external dependencies.
- `/lint-related-work` describes a two-phase fairness check (mechanical red flags + optional baseline cross-validation). Fork users implement the grep-and-compare logic; the dictionary of absolutist wording provided in the spec is a starter set to curate for each domain and language.

**Not included** (intentionally):

- Paper PDFs and extracted-text files (copyright)
- Paper screenshots / pasted images (copyright)
- The author's research notes, wiki sources, concept pages, synthesis pages, seminar reports, and PPT drafts — those live locally and are gitignored
- Personal Claude Code session history (`.claude/sessions/`) and permission-allowlist (`.claude/settings.json` — full of personal file paths)
- Obsidian workspace state and third-party plugin binaries (reinstalled from `community-plugins.json` on first launch)
- Obsidian personal preferences: `.obsidian/appearance.json` (theme, font size, light/dark mode), `.obsidian/graph.json` (graph-view layout, colour grouping, zoom), and `.obsidian/core-plugins.json` (which Obsidian built-in plugins are enabled or disabled). These are personal taste, not workflow configuration — each user picks their own, and Obsidian regenerates sensible defaults on first launch.

## Non-goals

- **Zero-config paper summaries.** If you want to drop PDFs into a hopper and get auto-summaries, this is not it. The review gate exists because quality matters more than throughput for research work; expect to spend 10–30 minutes per paper on `/review-note`.
- **A better Zotero.** No literature discovery, no citation export, no library sync. Pair with Zotero: Zotero for finding and filing papers, this repo for reading them deeply and synthesising across them.
- **A generic Obsidian starter.** The schema is tuned to academic papers (source / concept / entity / synthesis pages, `review_status` frontmatter, cascade-caveat citations). Adapt it for non-paper knowledge if you want, but expect to rewrite several commands.
- **Cloud backup or collaboration.** Everything is local files plus Claude Code. No hosted index, no user accounts, no multi-user sync. Pair with your own backup tool (OneDrive, Dropbox, git, etc.) — note that OneDrive and `.git/` do not play well together, so git separately from your vault sync path.

## Credit and license

The three-layer (raw / wiki / schema) architecture and the ingest / query / lint operations come directly from Andrej Karpathy, [*LLM Wiki: Pattern for Personal Knowledge Base Building*](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f). The review-gatekeeping layer, cascade-caveat rules, and paper-to-presentation commands are local extensions.

MIT — see [`LICENSE`](LICENSE).
