---
name: lint-related-work
description: Scan a paper note or PDF's Related Work section for framing issues (absolutist wording, uncited claims, scope mismatches) and optionally cross-validate against baseline papers
---

Automated fairness check for Related Work framing. Two phases: a fast mechanical pass and an optional cross-validation pass against baseline PDFs.

> **This repo ships the command spec only.** The implementation is intentionally not bundled; fork users either reimplement the checks described below or wrap their preferred linter under this command name.

## Use cases

1. **Reading a new paper**: `$ARGUMENTS` is a note or PDF path. Flag framing red flags immediately after `/read-paper`.
2. **Writing your own paper**: `$ARGUMENTS` is your draft path. Self-check for unfair framing before submission.
3. **Preparing a lab-seminar talk**: lint the paper's main baselines before the talk.

## Input format

```
/lint-related-work <path-to-note-or-pdf>
```

- `.md` note: read the Related Work or Problem-statement section and check every baseline description.
- `.pdf`: extract with `pdftotext -layout`, then locate the "Related Work" / "State of the Art" / "II." section.
- No argument: scan the current note.

## Checks

### Phase 1: mechanical red flags (fast)

1. **Absolutist-wording grep**: search the Related Work and baseline-description text for:
   - Completeness claims: `completely overlooked`, `entirely fails`, `cannot support`, `never`, `always fails`
   - Exaggeration claims: `significant burden`, `excessive`, `severely limits`, `critically flawed`

   Output: line number, quoted text, risk grade.

2. **Baseline extraction**: grep for `Author et al. [N]` patterns and list every cited baseline.

3. **Scope keyword check** (when input is a `.md` note):
   - Pull the paper's own keywords from frontmatter `tags` or Index Terms.
   - For each baseline, if its topic clearly does not match (e.g. an authentication paper citing a task-scheduling paper as a baseline), flag it.

### Phase 2: cross-validation against the baseline PDF (slow, requires baseline in `raw/papers/`)

4. **Self-description check** (needs baseline PDF):
   - For any baseline criticised with absolutist wording in Phase 1, look for its PDF in `raw/papers/`.
   - Extract the baseline's Abstract + Contributions sections.
   - Grep the baseline for the criticised claim.
   - Example: the paper criticises a baseline as "centralized", but the baseline's Abstract describes itself as "distributed" — severe inconsistency.

5. **Taxonomy check** (needs baseline PDF):
   - If the baseline has its own taxonomy table, compare it with the target paper's classification of the baseline.

## Output format

```markdown
## Related Work Lint Report — <paper name>

**Date**: <YYYY-MM-DD>
**Scope**: <section / line range>
**Sampled baselines**: <N>

### Phase 1 Red Flags

#### [RED] Absolutist wording (N hits)
- Line XX: "completely overlooked privacy" -- refers to [Author YYYY]
- Line YY: "entirely fails to consider..." -- refers to [Author YYYY]

#### [YELLOW] Possible scope mismatch (N hits)
- [Author YYYY]: topic is A (e.g. task scheduling), criticised for lacking B (e.g. privacy) -- manual check recommended.

#### Baselines (list)
- [26] Author et al. YYYY -- criticism: ...
- [27] Author et al. YYYY -- criticism: ...

### Phase 2 Cross-Check (when baseline PDF is in `raw/papers/`)

#### [OK] Verified fair (N baselines)
- [XX] -- baseline self-description matches the paper's criticism.

#### [WARN] Inconsistency found (N baselines)
- [XX] -- baseline declares "Direct Auth"; paper criticises "TA over-participation".
- [YY] -- baseline Abstract explicitly states "minimise cost"; criticism about privacy is out of scope.

### Suggested action

| Severity | Suggestion |
|---|---|
| Any red flag | Manual cross-check; treat as a prompt to investigate, not a verdict |
| Verified unfair | Downgrade the paper note to `reviewed_with_notes`; record a citation-fairness caveat |
| Verified strawman | Downgrade to `disputed` or attach a prominent warning wherever the paper is cited |
```

## Implementation notes

### Absolutist-wording dictionary (extensible)

Starter set (curate for your domain and languages):

```
completely overlooked
entirely fails
entirely overlooks
cannot support
never considers
always fails
significant burden
excessive overhead
severely limits
critically flawed
fundamentally unable
wholly inadequate
```

### Baseline pattern

```
([A-Z][a-z]+ (et al\.|and [A-Z][a-z]+) )?\[\d+\]
```

### Scope-mismatch heuristic

- If the paper's `tags: [authentication, cross-domain]` but a baseline's `tags: [task-scheduling, optimisation]` with no authentication tag: possible scope mismatch.
- If the baseline PDF has `security / authentication / privacy / anonym` occurrence count < 5: almost certainly not an authentication paper, and being criticised as one is likely a strawman.

## Reporting

1. Return a structured markdown report to the user.
2. Append a log entry to `wiki/log.md` when issues are found: `lint-related-work | <paper> -- N red flags`.
3. Suggest (do not auto-apply) a `review_status` downgrade when severe issues are found.

## Limits

- Lint only does mechanical checks; whether a criticism is actually unfair is human judgment.
- Phase 2 requires the baseline PDF in the vault; without it, only Phase 1 runs.
- The absolutist-wording dictionary needs ongoing curation.
- Supports source papers in any language as long as the dictionary is localized accordingly.

## Recommended cadence

- Run Phase 1 immediately after `/read-paper` on every new paper.
- Run Phase 2 on primary baselines before preparing a lab-seminar talk.
- Run before `/ingest-to-wiki` on any paper being promoted into the wiki.

## Notes

- Do not downgrade a paper based on lint output alone — every red flag needs manual confirmation.
- Absolutist wording is not always a problem (sometimes the criticism is genuinely well-founded); what matters is whether supporting evidence exists.
- The scope-mismatch heuristic can misfire; defer to the baseline's stated scope.
- When issues are found, report to the user for a decision — do not batch-downgrade notes.
