---
name: paper-report
description: Produce a seminar-report draft from a paper note and its PDF.
---

## Review-gate check (soft)

Before running, check `review_status`:

- `unreviewed`: warn — "This note has not been audited. A seminar report built on top of it may inherit errors. Running `/review-note` first is strongly recommended. (The seminar-report flow overlaps heavily with the audit flow anyway, so this is a good time to do both.)"
  - If the user continues, prepend: `> [UNREVIEWED] This report is based on an unreviewed note. Treat the PDF as the final authority.`
- `disputed`: continue, but the critical-analysis section **must** surface the specific concerns (e.g. perfect arithmetic progressions, in-text/table contradictions)
- `reviewed_with_notes`: quote the existing `review_notes` concerns in the critical-analysis section
- `reviewed`: run normally

**Tip**: The seminar-report flow (especially step 2 "claim-by-claim PDF check") overlaps heavily with `/review-note`. **Treat seminar preparation as a deep-audit opportunity** — once done, the user can upgrade `review_status` to `reviewed_with_notes` / `disputed` (if issues were found) and sync findings to the note's `## Review log` and to `wiki/review-history.md`.

## Main flow

Read the PDF I specify plus the paper note I have open, then write a seminar-report draft.

### Step 1: Load sources

- Read the full PDF content
- Read the comments and reactions in the note
- Use the PDF as the primary information source; use the note as commentary

### Step 2: Write the report

Structure:

1. Background and motivation
2. Related work (brief summary of 2–3 related studies)
3. System model and problem definition
4. Proposed method (core architecture and algorithm)
5. Experimental results and analysis
6. Critical analysis (strengths, weaknesses, logical-consistency check)
7. Relation to my research
8. Conclusion

### Step 3: Build links

- Use `[[wikilinks]]` in the new note to connect related notes
- Add reverse links in those related notes

## Requirements

- Use the output language specified in SKILL.md (keep technical terms in their original form).
- The critical-analysis section must have depth — concretely state whether motivation and methodology align.
- Preserve and integrate the user's personal critique from the source note.
- Where possible, suggest how the method could extend to the user's own domain.
- File name for the new note: `seminar-report-{paper-slug}-{YYYY-MM-DD}.md` at vault root.
