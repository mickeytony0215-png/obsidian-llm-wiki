---
name: review-note
description: Audit a paper note against the PDF; full check of claims, numbers, and logic; update `review_status` frontmatter and record the audit history.
---

# /review-note — paper-note audit

Audit whether a paper note is consistent with its PDF, so the note can be safely cited by the wiki.

## When to use

- Right after `/read-paper`, to upgrade the note from `review_status: unreviewed`
- When `/ingest-to-wiki`'s gate blocks you
- For re-auditing an existing note after a concern arises

## Review depth: full

Not a skim. Every claim is checked against the PDF, with particular attention to:
- Every quantitative datum (table cells, percentages, ms / Mbps / iteration counts)
- Core-claim consistency (motivation, problem, method, results, conclusion)
- Logical consistency (motivation → methodology → experiment → results → conclusion)
- Data credibility (perfect arithmetic progressions, in-text/table contradictions, "every metric is optimal" anti-patterns)

## Procedure

### Step 1: Load sources

1. Read the target paper note (usually a root-level `.md`)
2. Read its paired PDF (follow the `PDF source` link in frontmatter or the top-of-note link)
3. If the PDF path is missing, ask the user to confirm the actual location

### Step 2: Decompose claims

Walk through the note and list every claim, categorized as:

| Category | Example |
|---|---|
| Motivation / Problem | "Existing methods fail in condition X because Y." |
| System model | "Setup: 100 agents on a grid topology, 10-second simulation horizon." |
| Method core | "Method decomposes into three stages: A, B, C." |
| Math | "Metric_1 = f(x) / sd", "Score = Metric_1 / Metric_2" |
| Experiment setup | "Python 3.11, 1000-iteration simulation, 50k samples split 8:2 train/test" |
| Data point | Table 3: metric 9.47% → 7.11% → 4.74% → 2.37% |
| Conclusion | "The proposed method dominates all baselines on every metric." |

### Step 3: Claim-by-claim PDF check

For each claim, locate its passage in the PDF and compare:

| Marker | Meaning | Action |
|---|---|---|
| `[OK]` consistent | Exactly matches the PDF | Record as passed |
| `[DIFF]` minor inconsistency | Note says X, PDF says Y (imprecise but close) | List the delta, ask the user whether to edit the note or keep it |
| `[FAIL]` error | Note contradicts the PDF | Recommend the correction and explain the correct value |
| `[?]` not verifiable | The claim is in the note but has no PDF basis (may be the user's own addition) | Ask the user for the source, or mark it as "user-added commentary" |

### Step 4: Logical consistency check

Apply the six-axis check:

| Axis | What to check |
|---|---|
| Motivation → Problem Definition | Does the problem definition actually address the gap in the motivation? |
| Motivation → Related Work | Do the related-work gaps support the motivation? |
| Related Work → Methodology | Does the method respond directly to those gaps? |
| Methodology → Experiment | Does the setup validate the method's claims? (e.g. claims scalability but tests a single node) |
| Related Work → Baselines | Are the baselines from related work actually included in the experiment? |
| Results → Conclusions | Are the conclusions supported by the data? Any over-claiming? |

### Step 5: Data-credibility red flags

Actively check the following anti-patterns:

- **Perfect arithmetic progression**: table cells following x, x−k, x−2k, x−3k (real simulations almost never produce this)
- **Fixed offset**: metric-pair differences (e.g. Accuracy − Precision) identical across all models
- **All metrics optimal**: the proposed method wins on every metric (a missing trade-off is suspicious)
- **In-text vs. table mismatch**: prose says "rises" but the table falls; numbers disagree
- **Experimental scope vs. motivation mismatch**: claims high-speed but only simulates low-speed

### Step 6: Produce the Review Checklist

Return a structured checklist for the user to confirm item-by-item:

```markdown
## Review Checklist for <paper-slug>

### Verified consistent (N items)
- [list suppressed; user only needs the count]

### Inconsistencies — user decision required (N items)
1. **[section]**: note says "X", PDF says "Y"
   - Options: (a) edit the note to Y (b) keep X with justification (c) flag as a caveat
2. ...

### Likely errors (N items)
1. **[section]**: note "X" contradicts PDF "Y"
   - Recommendation: correct to Y
2. ...

### Not verifiable from PDF (N items; may be user-added)
1. **[section]**: this claim has no PDF basis
   - Ask: source? mark as user addition?

### Data-credibility red flags (N items)
1. Table 3 values form a perfect arithmetic progression (deltas 2.36 / 2.37 / 2.37)
   - Caution: real experiments rarely produce this pattern; consider marking as disputed.

### Logical consistency check results
- Motivation → Problem Definition: [OK] / [DIFF] / [FAIL] (with notes)
- ... other axes
```

### Step 7: Edit the note per the user's decisions

1. Edit the paragraphs the user chose to change
2. Append a `## Review log` section at the bottom:
   ```markdown
   ## Review log

   - **Review date**: YYYY-MM-DD
   - **Reviewer**: <user or delegate>
   - **Findings**:
     - [issue 1 summary]
     - [issue 2 summary]
   - **Final state**: reviewed / reviewed_with_notes / disputed
   - **Follow-ups**: [if any]
   ```

### Step 8: Update frontmatter

```yaml
review_status: reviewed          # or reviewed_with_notes / disputed
reviewed_date: 2026-04-18
reviewed_by: <user>
review_notes: "<one-line summary of concerns; may be empty for clean reviewed>"
```

### Step 9: Append to `wiki/review-history.md`

Format (newest first):

```markdown
## [YYYY-MM-DD] review | <paper-slug> — <state>

**File**: [[paper-note-name]]
**Reviewer**: <user>
**Final state**: reviewed / reviewed_with_notes / disputed

### Findings
- [issues found]
- [data-credibility red flags]
- [logical-consistency issues]

### Edits
- [changes made to the note]

### Follow-ups
- [e.g. retroactive check of wiki citations]
```

### Step 10: Cascade check if the note is already cited by the wiki

```bash
# Grep for all wiki pages that cite this note
grep -r "[[paper-note-name]]" wiki/
```

If citations exist, ask the user: "This note is already cited by these wiki pages: [list]. If the review found errors, do you want to fix those wiki pages too?"

If yes, prioritize:
1. `wiki/sources/` page (must be fixed; sync the `## Core contribution` / `## Key findings` sections)
2. `wiki/concepts/` page (if a cited number was wrong, fix or add a warning marker)
3. `wiki/synthesis/` page (check whether the error affects the conclusion)

Append each fix to `wiki/log.md` as: `## [YYYY-MM-DD] review-cascade | <paper>`

## Output

When finished, report:
- N items found (consistent / minor diff / error / unverifiable / red flag)
- Final `review_status`
- Files modified (the note itself, wiki citations, review-history, log)
- If `disputed`: list the specific cautions for future use of this note

## Notes

- One note at a time (batching degrades audit quality).
- The user can interrupt at any point (e.g. to check an external reference).
- If the PDF cannot be read (scanned, no OCR), report back and downgrade to a partial review; mark `review_status: disputed` with the reason.
- **Never auto-decide `reviewed`** — the user must confirm each item.
- **Never skip the data-credibility red-flag check**, even if the paper looks trustworthy.
