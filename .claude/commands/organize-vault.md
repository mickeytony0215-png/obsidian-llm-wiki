---
name: organize-vault
description: Scan the entire vault and build cross-links.
---

## Review-gate check (soft)

When scanning and encountering a note with `review_status: unreviewed`:

- In the final report, list these under a separate section: "Unreviewed notes (N)"
- When adding a link pointing **to** an unreviewed note, append an `[UNREVIEWED]` marker to the link
- For `disputed` notes, append a `[DISPUTED]` marker on links pointing to them
- At the end of the report, remind the user: "Run `/review-note` on each of these N unreviewed notes. After review, the `[UNREVIEWED]` markers can be cleared by grep or by the next run of `/organize-vault`."

**Do not skip unreviewed notes** — build the links, but carry the visible marker.

## Main flow

Scan every note in the vault and perform:

### Step 1: Inventory

List all notes in the vault, grouped by: paper notes, concept notes, seminar reports, other.

### Step 2: Backfill frontmatter

For every paper note missing YAML frontmatter, add `title`, `authors`, `year`, `venue`, `tags`, `status`.

### Step 3: Build bidirectional links

Analyze the contents of all notes and find related pairs, then add `[[links]]` in both directions.

Relatedness criteria:
- Same method or technique
- Same problem
- Same referenced literature
- Same sub-field

### Step 4: Create missing concept notes

Extract recurring core concepts from all paper notes. Create a concept note if one does not already exist.

### Step 5: Produce the report

On completion, list:
- Which notes were modified
- Which concept notes were added
- How many new links were created

## Requirements

- Use the output language specified in SKILL.md (keep technical terms in their original form).
- Do not delete existing content — only add links and frontmatter.
