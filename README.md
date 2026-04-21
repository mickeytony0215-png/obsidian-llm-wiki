# obsidian-llm-wiki

An Obsidian + Claude Code workflow for reading academic papers and building a personal research wiki. This repo ships **only the workflow** — the slash commands, the schema, the Obsidian config — without the author's personal research notes. Clone it, drop in your own PDFs, and you get a reproducible pipeline.

The pipeline follows Andrej Karpathy's [LLM wiki pattern](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) (three-layer architecture: raw / wiki / schema; three operations: ingest / query / lint), with two extensions: a **review-gatekeeping layer** that prevents unreviewed notes from polluting the wiki, and a **paper-to-presentation pipeline** that turns reviewed notes into seminar drafts and slide material.

## Contents

1. [The three layers](#the-three-layers)
2. [The ten slash commands](#the-ten-slash-commands)
3. [Typical workflow](#typical-workflow)
4. [The review gate](#the-review-gate)
5. [Setup](#setup)
6. [What is and isn't in this repo](#what-is-and-isnt-in-this-repo)
7. [Credit and license](#credit-and-license)

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
  └── commands/           # The ten slash commands documented below
note/Templates/           # Obsidian note templates for new papers
.obsidian/                # Vault config (Dataview, Claudian, BRAT)
```

## The ten slash commands

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
4. **The ten slash commands are already wired up** — just type `/read-paper` inside Claude Code while the vault is the working directory. `SKILL.md` is loaded as shared context automatically.
5. **Edit `SKILL.md`** at `.claude/skills/SKILL.md`. The top section ("Researcher profile") has placeholders for your role, research domain, output language, and common obligations — fill them in so Claude tailors analysis and output accordingly. Everything below the horizontal rule is the universal workflow spec and should not need editing.

### Recommended customization points

- **`SKILL.md`** — research domain, output language, preferred tools. This is the single biggest lever.
- **`note/Templates/paper-template.md`** — the frontmatter schema for new paper notes.
- **`wiki/index.md`** — Dataview queries. Edit if you rename or add wiki subfolders.
- **Command files under `.claude/commands/`** — tweak trigger phrasing, output format, or review-gate strictness.

## What is and isn't in this repo

**Included**: all ten slash commands, `SKILL.md`, the wiki schema (`index.md`, `dashboard.md`), Obsidian config (app settings, plugin list, graph settings), Obsidian note templates, and this README.

**Not included** (intentionally):

- Paper PDFs and extracted-text files (copyright)
- Paper screenshots / pasted images (copyright)
- The author's research notes, wiki sources, concept pages, synthesis pages, seminar reports, and PPT drafts — those live locally and are gitignored
- Personal Claude Code session history (`.claude/sessions/`) and permission-allowlist (`.claude/settings.json` — full of personal file paths)
- Obsidian workspace state and third-party plugin binaries (reinstalled from `community-plugins.json` on first launch)

## Credit and license

The three-layer (raw / wiki / schema) architecture and the ingest / query / lint operations come directly from Andrej Karpathy, [*LLM Wiki: Pattern for Personal Knowledge Base Building*](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f). The review-gatekeeping layer, cascade-caveat rules, and paper-to-presentation commands are local extensions.

MIT — see [`LICENSE`](LICENSE).
