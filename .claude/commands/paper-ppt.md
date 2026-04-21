---
name: paper-ppt
description: Build a seminar-slide draft for a paper (English bullets + speaker notes in the user's preferred language); goal is to help an audience understand the paper.
---

Build a **seminar slide-draft note** for a paper. Goal: help faculty and peers grasp the paper's **core contribution**, **method logic**, **experimental results**, and **critical perspective**.

## Language rule

| Content | Language |
|---|---|
| Slide bullets / table headers / captions | **English** (each bullet < 15 words) |
| Speaker notes | The preferred output language from SKILL.md's Researcher profile (keep established technical terms in their original form — do not translate loanwords) |
| Warning notes (for `disputed` / `unreviewed`) | The preferred output language |

## Review gate

| status | Handling |
|---|---|
| `reviewed` | Produce normally |
| `reviewed_with_notes` | The Pros & Cons slide **must** cite `review_notes` concerns |
| `disputed` | **Mandatory**: Slide 1 gets a dispute warning + Pros & Cons slide details every concern + Conclusion slide adds "borrow the idea, not the numbers" |
| `unreviewed` | Warn; if the user continues, prepend "based on an unreviewed note" in the draft header |

## Output file

Path: `slide-draft-{paper-slug}-{YYYY-MM-DD}.md` at vault root.

## Frontmatter

```yaml
---
title: "Slide draft — {paper-slug}"
date: YYYY-MM-DD
type: slide-draft
source: "[[paper-note]]"
review_status_of_source: {status}
tags: [slide-draft, seminar, ...]
---
```

## Per-slide format (minimalist)

**Default (bullet slides)**:

```markdown
## Slide N — English topic

**Slide**:
- Short English bullet
- Another English bullet
- 3–5 bullets per slide

**Speaker notes**: Narrative in the preferred output language, adding context the bullets skip. ~150–300 chars, conversational, not paper verbatim.
```

**Exception 1 (title slide)**:

```markdown
## Slide 1 — Title

**Slide**:
> **{Paper full title}**
>
> Authors: {authors}
> Venue: {venue}, {year}
>
> Date: YYYY-MM-DD

**Speaker notes**: Opening line introducing the paper, authors, venue, and a one-sentence core claim, in the preferred output language.
```

**Exception 2 (architecture diagram — keep ASCII / figure)**:

```markdown
## Slide 6 — System architecture

**Slide**:
\```
[ASCII architecture diagram]
\```

**Speaker notes**: Explain what this diagram shows, in the preferred output language.
```

**Exception 3 (data table — keep the markdown table)**:

```markdown
## Slide 13 — Computation overhead

**Slide**:

| Scheme | Initial AKE | Handovered AKE |
|---|:---:|:---:|
| ... | ... | ... |

**Speaker notes**: Explain what the table compares, in the preferred output language.
```

**Exception 4 (math formula — keep LaTeX)**:

```markdown
## Slide 10 — Session-key formula

**Slide**:

$$SK = KDF([t_{E}] \cdot T_{MN})$$

where $T_{MN} = [t_{MN}] \cdot G$

**Speaker notes**: Explain the meaning, in the preferred output language.
```

## Default structure (14–18 slides, adjustable per paper)

| Slide | English topic |
|:-:|---|
| 1 | Title |
| 2 | Outline |
| 3 | Background & Motivation |
| 4 | Research Gap / Problem Statement |
| 5 | Related Work |
| 6–7 | System Model (incl. architecture) |
| 8–10 | Proposed Method (algorithm / flow / formulas) |
| 11 | Experimental Setup |
| 12–13 | Results (tables / figures) |
| 14–15 | Pros & Cons / Critical Analysis |
| 16 | Relation to My Research |
| 17 | Conclusion & Future Work |
| 18 | Q&A |

## Disputed-paper mandatory warning

**Slide 1 addition**:

```markdown
> [DISPUTED] Data-credibility concerns
> - Perfect arithmetic progressions in tables
> - In-text/table inconsistencies
> - (or other specific issues from review_notes)
>
> Framing: borrow the idea, critique the numbers.

**Speaker notes**: Open by openly acknowledging that this paper is disputed — we present it to demonstrate critical reading. (In the preferred output language.)
```

**Pros & Cons slide must list**: each concern as an English bullet + detailed speaker notes.

**Conclusion slide must add**: `Borrow the idea, not the numbers.`

## "Related links" section (end of the draft)

```markdown
## Related links

- Paper note: [[paper-note]]
- Wiki source: [[wiki/sources/xxx]] (if ingested)
- Prior seminar report: [[seminar-report-xxx]] (if any)
- Related concepts: [[concepts/xxx]]
- Related synthesis: [[synthesis/xxx]] (if any)
```

## Requirements

- Bullets are tight (**< 15 words each**, English)
- Speaker notes sound natural (like talking to a classmate, not reading the paper)
- Speaker notes fill in "why this design choice" and "how this differs from paper X"
- If disputed / with concerns, **raise them openly**
- **Must include Slide 16 (Relation to My Research)**
- Speaker notes are **not full English sentences** (technical terms in English are fine)

## Forbidden

- Bullets longer than 15 words
- Copy-pasting paragraphs verbatim from the paper into bullets
- Speaker notes that are word-for-word identical to the bullets (speaker notes supplement, not restate)
- Skipping "Relation to My Research"

## Triggers

- "Build the PPT content for <paper-slug>"
- "Make slide material for <paper-slug>"
- "/paper-ppt <paper-slug>"
- "Build a seminar slide draft for [paper]"
