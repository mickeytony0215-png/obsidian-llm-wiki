---
name: paper-summary
description: Analyze a paper and fill in a structured summary in the note.
---

## Review-gate check (soft)

Before running, check `review_status`:

- `unreviewed`: warn — "This note has not been audited. The summary may inherit errors. Running `/review-note` first is recommended. Continue?"
  - If the user continues, prepend to the summary: `> [UNREVIEWED] This summary is produced from an unreviewed note and may contain errors. After review, this banner can be removed.`
- `disputed`: continue, but prepend: `> [DISPUTED] This paper has known concerns (see review_notes). The summary is for reference only.`
- `reviewed` / `reviewed_with_notes`: run normally

**Note**: this skill must not change `review_status` (only `/review-note` can do that). If you encounter content that disagrees with the PDF while running, mark the passage with `[VERIFY]` in the note and notify the user.

## Main flow

Analyze the paper note I have open and fill in the following sections:

1. Research question — one paragraph: what problem does this paper solve?
2. System model / scenario assumptions — list the key assumptions
3. Proposed method — describe the core algorithm / framework
4. Key technical details — formulas and optimization objective
5. Strengths and weaknesses — give concrete critique, not "room for improvement"
6. Relation to my research — connect to the user's domain from SKILL.md

Use the output language specified in SKILL.md (keep technical terms in their original form). Write directly into the note's corresponding sections.
