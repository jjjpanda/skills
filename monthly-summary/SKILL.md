---
name: monthly-summary
description: Generate a monthly summary from Obsidian daily and weekly notes. Uses the previous complete month when invoked early in a new month; otherwise summarizes the current month to date. Confirms date range before summarizing.
---

Generate a monthly summary from Obsidian periodic notes.

## Step 1: Determine Date Range

Get today's date from the system context.

Apply this rule to pick the target month:
- **Day 1–7 of the month**: default to the **previous complete month** (e.g., Feb 3 → all of January)
- **Day 8+**: default to the **current month so far** (e.g., Jan 30 → Jan 1–30)

State the range clearly before proceeding:
> "Summarizing notes from **[Month 1st]** through **[last day of target month or today]** — is this correct?"

Wait for confirmation. If the user corrects the range, use their specified dates instead.

## Step 2: Locate & Read Notes

Paths:
- Daily notes: `doc/notes/periodic/daily/YYYY-MM-DD.md`
- Weekly notes: `doc/notes/periodic/weekly/`
- Monthly notes: `doc/notes/periodic/monthly/`

Read the monthly note for the target period if it exists. Glob all weekly and daily files whose dates overlap the range and read them; use `ai_summary` frontmatter if present on weeklies. Read the previous monthly note for continuity context.

For each note, extract wikilinks (`[[...]]`). Resolve each linked file relative to the vault root and read it for additional context.

Source priority: monthly note first (most structured), then weeklies, then dailies for dates and details not captured above.

## Step 2b: Semantic Context Discovery

Use the **query-vault** skill (read `**/query-vault/SKILL.md`). Queries must be **entity names likely to have their own note**: specific possessions, named locations physically visited, named project docs, named products. Do not query for event descriptions or themes (e.g. "bill payment", "weekend plans", "bug fix" — these return irrelevant hits). Use confirmed results as additional context and as wikilink candidates for the output.

## Step 3: Generate Summary

Style — **no preamble, no closing paragraph**:

```
## AI Summary

### [theme]
- [event]
  - [sub-item if needed]

### [next theme]
- ...
```

Rules:
- Themes (`###`) emerge from the notes — pick whatever domains had activity. Don't use a fixed list.
- Include every distinct event. Do not self-curate or editorialize. Prefer inclusion over omission.
- Terse, lowercase, first-person implied. Personal voice and brief opinions allowed when they carry signal.
- **Named notes as contextual links**: when an entity has its own note (possession, location, project), wikilink to that note — display text should fit naturally in the phrase.
  - ✓ `working on [[Side Project Notes|side project]] [[Data Pipeline|data pipeline]] runs`
  - ✓ `[[Old Laptop]] set up for couch use`
  - ✓ `car at [[Local Auto Shop]]`
- **Dates as contextual anchors**: when an entity has no dedicated note, anchor it to its date instead. Embed `[[YYYY-MM-DD]]` mid-phrase with descriptive display text. Do not trail bullets with `on [[date]]`. A bullet must never end in a parenthetical date. If you find yourself about to write `(...[[date]]...)` at the end of a line, stop and rewrite the sentence so the date is aliased and moved earlier in the phrase.
  - ✓ `[[2026-04-07|experimented]] with local llms`
  - ✓ `[[Old Laptop]] set up for [[2026-04-12|couch use]]` — date anchors the moment, named note links the possession
  - ✗ `experimented with local llms on [[2026-04-07]]`
  - ✗ `event happened ([[2026-06-06]])` — trailing bare date in parens
- **Multi-day spans**: use `[[date1]]–[[date2]]` embedded mid-phrase, same anchor rules as above.
  - ✓ `watched mission impossible 1–5 [[2026-04-20]]–[[2026-04-26]]`
- First names only for people. No wikilinks on people's names.
- Nest sub-bullets for sub-components of a parent event.
- Do not group by week.

## Step 4: Self-Check

Before finalizing, re-read the draft and fix:
- **Trailing parenthetical dates**: scan for `([[` on any line ending in `]])`. Fix every hit by aliasing the date and moving it earlier in the phrase per the "Dates as contextual anchors" rule.
- **Em-dashes**: remove every `—`. Split into two bullets, nest a sub-bullet, or reword. The user does not write with em-dashes.
- **Non-natural AI language**: strip corporate/LLM tells — no "leveraged", "delved into", "streamlined", "showcased", "navigated", "fostered", "seamless", "robust", "underscored", inflated verbs, or hedging. Match the user's terse, lowercase, first-person voice.
