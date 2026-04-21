---
name: ingest-to-wiki
description: Integrate the paper/article under discussion into the wiki — create a source page and update related concept/entity pages and the index. Must pass the review gate first.
---

Identify the paper or article discussed in the current conversation, then follow the steps below to create a source summary page, update concept/entity pages, update `index.md`, and prepend a `log.md` entry. Process one source at a time.

## [HARD GATE] Step 0: review gate (mandatory)

**Before any ingest action, check the source note's `review_status`:**

1. Identify the source note (usually a root-level paper note)
2. Read its frontmatter `review_status` field
3. Act according to the state:

| review_status | Action |
|---|---|
| `unreviewed` or missing | **[HARD GATE] Stop immediately. Refuse to ingest.** Report: "This note's review_status is unreviewed. It cannot be integrated into the wiki. Run `/review-note <note-name>` first. The review gate exists to prevent unverified content from propagating into the wiki and causing hard-to-trace errors." |
| `reviewed` | Continue to the following steps |
| `reviewed_with_notes` | Continue, but **mandatory**: quote the caveat from `review_notes` inside the wiki source page's `## Personal notes` section |
| `disputed` | **Require explicit user confirmation**: "This note has known disputes (`review_notes`: ...). After ingest, the dispute will be clearly flagged on the wiki source page's `## Personal notes`. Continue?" Only proceed on confirmation, and always flag the dispute. |

**Never skip this gate** — even if the user directly says "ingest this" without reviewing, refuse and redirect to `/review-note`.

## Step 1: Create the source page

Create the page under `wiki/sources/` in the schema format; frontmatter must include `summary:` and `venue_short:`.

If the source note's `review_status` is `disputed`, the `## Key findings` section of the source page must begin with:

```markdown
> [DISPUTED] data credibility: this paper was flagged during review for [issue summary]. When citing data from this page, cross-reference the critical analysis in `## Personal notes`.
```

## Step 2: Scan related pages

Scan `wiki/index.md` (Dataview output) or grep for related existing concept, entity, and source pages.

## Step 3: Update or create concept pages

- New pages: frontmatter must include `summary:` and `source_count:`
- Existing pages: when adding this source, if the source is `disputed` or `reviewed_with_notes`, append a caveat next to the citation, for example:
  ```markdown
  - [[sources/xxx]] — proposes Y ([DISPUTED] data credibility)
  ```

## Step 4: Update or create entity pages

New pages: frontmatter must include `summary:` and `type:`.

## Step 5: index.md

`index.md` auto-syncs via Dataview; no manual edits needed — just make sure the new page's frontmatter fields are complete.

## Step 6: Prepend a log entry

Prepend (reverse chronological) a new entry in `wiki/log.md` — the entry goes at the top of the log, right after the convention header block and before the first existing `## [...]` entry.

Format example:

```markdown
## [2026-04-18] ingest | <author-year> — <short-name>

- Source-note review_status: reviewed / reviewed_with_notes / disputed
- Created source page: [[sources/xxx]]
- Updated concept pages: [[concepts/xxx]], [[concepts/yyy]]
- Created entity pages: [[entities/xxx]] (if any)
- New associations found: [...]
- If disputed: how the dispute is flagged in the wiki
```

## Step 7: Report back

List which pages were created/updated and any new associations or contradictions discovered. If the source note is `reviewed_with_notes` or `disputed`, summarize how the caveats/disputes are presented in the wiki.

## Cascade caveat rules (for citations)

In the `Sources` section or at each citation inside `wiki/concepts/`, `wiki/entities/`, and `wiki/synthesis/` pages, mark every non-`reviewed` source:

| status | Citation marker |
|---|---|
| `reviewed` | `[[sources/xxx]] — proposes Y` (no marker) |
| `reviewed_with_notes` | `[[sources/xxx]] — proposes Y (note: see the Z caveat in review_notes)` |
| `disputed` | `[[sources/xxx]] — proposes Y ([DISPUTED]: specific reason; borrow the idea, not the numbers)` |
| `unreviewed` | `[[sources/xxx]] — proposes Y [UNREVIEWED: auto-generated note, PDF not yet audited]` |

These rules align with the "Cascade caveat rules" section in SKILL.md.

## Notes

- **One source at a time** — batching reduces human-in-the-loop review quality.
- Updating a concept page means **merge**, not overwrite — preserve prior sources' positions and mark disagreements.
- If a new source contradicts existing wiki claims, mark the disagreement on the concept page.
- Incorporate the user's personal thoughts and questions from the conversation.
- Missing frontmatter causes Dataview to render blank cells — always fill fields.
- **Never skip the review gate** — if the user insists on skipping, remind them that this pollutes the knowledge base, and refuse.

## Batch-ingest handling

If the user asks to ingest multiple papers at once (e.g. "ingest these six"), the review gate still applies **per paper**:

- Check every note's `review_status` first
- If any one is `unreviewed`, stop and require `/review-note` on that one before any ingest
- A safe pattern: finish reviewing all the candidates first, then ingest the batch
