---
name: organize-note
description: Rebuild a paper note from the PDF plus the user's existing rough notes.
---

Given a PDF and the user's current (rough) paper note, reconstruct a complete, structured paper note.

## Step 1: Load and compare

- Read the full PDF content
- Read the current note
- **Preserve every personal comment, critique, and reaction in the existing note — those are the most important content.**

## Step 2: Reconstruct

Rewrite the note into this standard structure:

- YAML frontmatter: `title`, `authors`, `year`, `venue`, `tags`, `status: done`
- Paper info (title, authors, year, keywords)
- Research question — add a complete problem description from the PDF
- System model / scenario assumptions — add concrete assumptions from the PDF
- Proposed method — add algorithmic detail from the PDF
- Key technical details — add formulas and optimization objectives from the PDF
- Experimental setup — add simulator, parameters, baselines from the PDF
- Main results — add the key data points from the PDF
- Strengths and weaknesses — **keep the user's original critique**, then add a concrete analysis from the PDF
- Relation to my research — **keep the user's original framing**, then add more linkages
- Key notes / reactions — **keep the user's original notes verbatim**
- Related concepts

## Step 3: Auto-link

- Scan every other note in the vault and find related ones
- Add `[[links]]` here, and add reverse links in those related notes

## Step 4: Concept promotion

- Extract core concepts; add links to existing concept notes; create new concept notes when needed

## Requirements

- Use the output language specified in SKILL.md's Researcher profile (keep technical terms in their original form).
- **Every existing personal comment must survive**, wrapped in a blockquote to distinguish from the reconstructed content.
- Overwrite the current note in place — do not create a new file.
- Report which notes were edited and which were added.
