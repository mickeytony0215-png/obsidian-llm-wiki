---
name: concept-extract
description: Extract core concepts from the current paper note and create atomic concept notes.
---

## Review-gate check (soft)

Before running, check the note's `review_status`:

- `unreviewed`: warn the user — "This note has not been audited. Extracted concepts may be based on a misreading of the paper. Running `/review-note` first is strongly recommended."
  - If the user continues:
    - Any new concept note gets `status: draft` in frontmatter (to mark that it may need updating after review)
    - When linking this paper from an existing concept note, append `[UNREVIEWED]` to the link
- `disputed`: skip the paper's contested claims when extracting; do not cite the suspect data on any new concept page
- `reviewed` / `reviewed_with_notes`: run normally (for `reviewed_with_notes`, consider adding a short caveat where appropriate)

## Main flow

Analyze the paper note I have open and extract 3–5 core concepts.

For each concept:

1. Check whether a matching concept note already exists in the vault
2. If not, create a new concept note in the appropriate folder
3. In the paper note, add a `[[wikilink]]` to each concept

Concept-note format:

- A short definition / description
- Links to the relevant papers
- Links to related concepts

Use the output language specified in SKILL.md (keep technical terms in their original form).
