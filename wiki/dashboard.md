---
title: "Wiki health dashboard"
tags: [meta, dashboard, dataview]
created: 2026-04-18
summary: "Live wiki health check: missing fields, orphan pages, recent activity"
---

# Wiki health dashboard

> This page is computed live by Dataview. No manual maintenance — it re-renders every time Obsidian opens or files change, replacing most of the manual lint work.
>
> Complements `/lint-wiki`: Dataview handles ongoing **structural** checks; the lint skill handles **semantic** checks (stale claims, new concept promotion opportunities, cross-source contradictions).

---

## Review status (review gate)

> Paper notes must pass `/review-note` before being citable by the wiki. Unreviewed notes are hard-blocked from `/ingest-to-wiki` and soft-warned elsewhere. See [[pending-review]] for the full list.

### Status distribution

```dataview
TABLE WITHOUT ID
  review_status AS "Status",
  length(rows) AS "Count"
FROM "."
WHERE contains(file.tags, "#paper") AND review_status
GROUP BY review_status
SORT review_status ASC
```

### High-priority pending review (5 most recent `unreviewed`)

```dataview
TABLE WITHOUT ID
  file.link AS "Note",
  default(year, "-") AS "Year",
  default(venue, "-") AS "Venue"
FROM "."
WHERE contains(file.tags, "#paper") AND review_status = "unreviewed"
SORT file.mtime DESC
LIMIT 5
```

### Disputed notes

> These notes must carry an explicit dispute marker wherever the wiki cites them.

```dataview
TABLE WITHOUT ID
  file.link AS "Note",
  review_notes AS "Concern summary",
  reviewed_date AS "Review date"
FROM "."
WHERE contains(file.tags, "#paper") AND review_status = "disputed"
SORT reviewed_date DESC
```

### Papers tagged but missing `review_status` (gatekeeping leak)

```dataview
TABLE WITHOUT ID
  file.link AS "Note",
  file.mtime AS "Last modified"
FROM "."
WHERE contains(file.tags, "#paper") AND !review_status
SORT file.mtime DESC
```

---

## Synthesis pages (cross-source analyses)

```dataview
TABLE WITHOUT ID
  file.link AS "Page",
  summary AS "Topic",
  source_count AS "Cited sources",
  created AS "Created"
FROM "wiki/synthesis"
SORT created DESC
```

## Slide drafts (seminar material)

```dataview
TABLE WITHOUT ID
  file.link AS "Draft",
  source AS "Source paper",
  review_status_of_source AS "Source review status",
  date AS "Produced"
FROM "."
WHERE type = "slide-draft"
SORT date DESC
```

---

## Pages with missing frontmatter fields

### Every page should have a `summary` field (Dataview dependency)

```dataview
TABLE WITHOUT ID
  file.link AS "Page",
  file.folder AS "Section"
FROM "wiki/concepts" OR "wiki/entities" OR "wiki/sources" OR "wiki/synthesis"
WHERE !summary
```

### Source pages missing `venue_short`

```dataview
TABLE WITHOUT ID
  file.link AS "Page",
  venue AS "Full venue"
FROM "wiki/sources"
WHERE !venue_short
```

### Concept pages missing `source_count`

```dataview
TABLE WITHOUT ID
  file.link AS "Page",
  length(file.inlinks) AS "Inbound links (reference only)"
FROM "wiki/concepts"
WHERE !source_count
```

### Source pages missing `year`

```dataview
TABLE WITHOUT ID
  file.link AS "Page"
FROM "wiki/sources"
WHERE !year
```

---

## Orphan pages (no inbound links)

> "Orphan" means: no vault page outside of `index.md`, `log.md`, this `dashboard.md`, or `wiki/` meta pages links to this page. The filter below excludes meta pages so they do not mask true orphans.

```dataview
TABLE WITHOUT ID
  file.link AS "Orphan page",
  file.folder AS "Section",
  summary AS "Summary"
FROM "wiki/concepts" OR "wiki/entities" OR "wiki/sources" OR "wiki/synthesis"
WHERE length(filter(file.inlinks, (link) =>
  !contains(string(link.path), "wiki/index") AND
  !contains(string(link.path), "wiki/log") AND
  !contains(string(link.path), "wiki/dashboard")
)) = 0
```

---

## Recent activity

### Latest ingested (top 5)

```dataview
TABLE WITHOUT ID
  file.link AS "Page",
  summary AS "Title",
  ingested AS "Ingested"
FROM "wiki/sources"
SORT ingested DESC
LIMIT 5
```

### Recently updated concept pages (top 5)

```dataview
TABLE WITHOUT ID
  file.link AS "Page",
  choice(last_updated, last_updated, updated) AS "Updated",
  source_count AS "Sources"
FROM "wiki/concepts"
SORT choice(last_updated, last_updated, updated) DESC
LIMIT 5
```

---

## Link hotspots

### Most-cited pages (vault hubs)

```dataview
TABLE WITHOUT ID
  file.link AS "Page",
  file.folder AS "Section",
  length(file.inlinks) AS "Inbound"
FROM "wiki/concepts" OR "wiki/entities" OR "wiki/sources" OR "wiki/synthesis"
SORT length(file.inlinks) DESC
LIMIT 10
```

### Pages that cite the most others (vault narrators)

```dataview
TABLE WITHOUT ID
  file.link AS "Page",
  file.folder AS "Section",
  length(file.outlinks) AS "Outbound"
FROM "wiki/concepts" OR "wiki/entities" OR "wiki/sources" OR "wiki/synthesis"
SORT length(file.outlinks) DESC
LIMIT 10
```

---

## Frontmatter naming consistency

### `updated:` vs `last_updated:` mixed use

> The vault convention is `last_updated:` (see the template in SKILL.md). The query below lists pages still using the legacy `updated:` field.

```dataview
TABLE WITHOUT ID
  file.link AS "Page",
  updated AS "Uses updated:"
FROM "wiki/concepts" OR "wiki/entities"
WHERE updated AND !last_updated
```

---

## Per-section quality average

```dataview
TABLE WITHOUT ID
  file.folder AS "Section",
  length(rows) AS "Pages",
  round(sum(map(rows, (r) => choice(r.summary, 1, 0))) / length(rows) * 100) AS "Summary completion %",
  round(sum(map(rows, (r) => choice(length(r.file.inlinks), 1, 0))) / length(rows) * 100) AS "Has inbound %"
FROM "wiki/concepts" OR "wiki/entities" OR "wiki/sources" OR "wiki/synthesis"
GROUP BY file.folder
SORT file.folder ASC
```

---

## How to read this dashboard

| Metric | Target | Why it matters |
|---|:-:|---|
| Pages missing `summary` | **0** | Dataview index renders blank cells otherwise |
| Orphan page count | **0** | Every page should have at least one non-meta inbound link, or it is "dead knowledge" |
| Sources missing `venue_short` | **0** | Otherwise the index's Venue column is blank |
| `summary` completion rate | **100%** | Every page should be identifiable at a glance |
| Unreviewed paper count | **0** (or an acceptable backlog) | Unreviewed notes cannot be cited; growing backlog means review is not keeping up with `/read-paper` |
| Papers tagged but missing `review_status` | **0** | Gatekeeping leak — the review gate cannot enforce itself |
| Recent ingest within 10 days | Depends on cadence | Long idle periods indicate wiki stagnation |

## Running the full lint

This dashboard does only **structural** checks. Semantic checks (stale claims, missed concept promotions, cross-source contradictions) still require:

```
/lint-wiki
```

Recommended cadence: every 5–10 new sources, or whenever this dashboard shows warnings.
