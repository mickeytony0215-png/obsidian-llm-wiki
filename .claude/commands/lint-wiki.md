---
name: lint-wiki
description: Health-check the wiki — find orphan pages, broken links, stale content, and cascade-caveat inconsistencies, then report improvements.
---

Execute the Lint procedure defined in SKILL.md. Scan all pages under `wiki/` and produce a health-check report.

## Checks (per SKILL.md "Lint" section)

### Structural checks (fast)

1. **Frontmatter completeness**: `summary:` / `venue_short:` / `source_count:` / `year:` (Dataview depends on them)
2. **Frontmatter naming consistency**: mixed use of `updated:` vs `last_updated:`
3. **Orphan pages**: no inbound `[[wikilink]]` (excluding meta pages: `log`, `index`, `dashboard`, `pending-review`, `review-history`, `integration-progress`, `SKILL`)
4. **Broken links**: `[[xxx]]` pointing to a non-existent page
5. **Source pages missing the "Relation to the knowledge base" section**

### Semantic checks (slow)

6. **Concepts mentioned in prose but not yet promoted to their own page**
7. **Concept pages whose "Current state" or "Key methods" sections may need updating** (new sources joined but the section was not updated)
8. **Contradictions or stale claims**
9. **Suggested next directions for reading or searching**

### Phase 2 (review cascade)

10. **Cascade-caveat consistency**: for each `disputed`, `reviewed_with_notes`, or `unreviewed` paper, confirm that every wiki citation carries the correct marker (see SKILL.md "Cascade caveat rules"):
    - `disputed` citations must carry `[DISPUTED]`
    - `unreviewed` citations must carry `[UNREVIEWED]`
    - `reviewed_with_notes` citations should carry a caveat when relevant (recommended, not strict)
11. **Review-status distribution**: counts of `reviewed` / `reviewed_with_notes` / `disputed` / `unreviewed`; if `unreviewed` exceeds 20 %, flag a cleanup reminder

## Output

1. **Report** the structured lint findings to the user (categories, counts, explicit lists)
2. **Auto-fix** mechanical issues (e.g. frontmatter naming inconsistency, missing cascade caveats)
3. **Append a lint entry to `wiki/log.md`** recording:
   - Scale statistics (pages per section)
   - Frontmatter completeness metrics
   - Issue categories and counts
   - Auto-fixed items
   - Items left for later

## Recommended cadence

- Every 5–10 new sources
- After any large batch-ingest
- Phase 2 cascade audit: monthly

## Notes

- Lint does **not** judge content quality — that is `/review-note`'s job.
- Lint performs only **structural / consistency / cascade-completeness** checks.
- When severe problems are found (e.g. many broken links), report first and let the user decide processing order — do not auto-edit at scale.
