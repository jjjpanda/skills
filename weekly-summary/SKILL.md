---
name: weekly-summary
description: Generate a weekly summary from Obsidian daily notes and other notes created in the past 7 days. Confirms date range before summarizing.
---

Generate a weekly summary from Obsidian daily notes.

## Step 1: Determine Date Range

Get today's date from the system context. Calculate the 7-day window: from `today - 6 days` through `today` (inclusive).

State the range clearly before proceeding:
> "Summarizing notes from **[start date]** through **[end date]** — is this correct?"

Wait for confirmation before continuing.

## Step 2: Locate & Read Daily Notes

- Daily notes: `doc/notes/periodic/daily/YYYY-MM-DD.md`
- Weekly notes: `doc/notes/periodic/weekly/`

Read through previous weekly note for context if it exists. This can help maintain continuity and capture ongoing themes.

Glob for each date in the range (format: `YYYY-MM-DD.md`). Read every file found; skip dates with no file.

For each daily note, extract any wikilinks (`[[...]]`). Resolve each linked file relative to the vault root and read it for additional context.

## Step 2b: Semantic Context Discovery

Use the **query-vault** skill (read `**/query-vault/SKILL.md`). Queries: entity names that are plausibly standalone note titles in this vault — specific possessions, specific locations physically visited, named project or codebase docs. Skip event descriptions, institution names, financial/legal terms, acronyms, and anything already resolved via wikilink. Use confirmed results as additional context and as wikilink candidates.

## Step 3: Generate Summary

Style — **no preamble, no closing paragraph**:

```
## Weekly Summary — [start date] to [end date]

### Highlights
- [most significant event]
  - [sub-item if needed]

### Themes & Patterns
- [recurring topic, ongoing project, or pattern]

### Open Items
- [task, follow-up, or deferred item]
```

Rules:
- Terse, lowercase, first-person implied. Personal voice and brief opinions allowed when they carry signal.
- Prioritize signal over exhaustive coverage.
- **Named notes as contextual links**: when an entity has its own note (possession, location, project), wikilink to that note — display text should fit naturally in the phrase.
  - ✓ `[[Keychron Q2]] set up`
  - ✓ `car at [[Local Auto Shop]]`
- **Dates as contextual anchors**: when an entity has no dedicated note, anchor it to its date instead. Embed `[[YYYY-MM-DD]]` mid-phrase with descriptive display text. Do not trail bullets with `on [[date]]`.
  - ✓ `[[2026-04-30|merged]] auth refactor`
  - ✗ `merged auth refactor on [[2026-04-30]]`
- **Multi-day spans**: use `[[date1]]–[[date2]]` embedded mid-phrase.
  - ✓ `auth refactor [[2026-04-28]]–[[2026-04-30]] merged`
- First names only for people. No wikilinks on people's names.
- Nest sub-bullets for sub-components of a parent event.

## Step 4: Self-Check

Before finalizing, re-read the draft and fix:
- **Trailing parenthetical dates**: scan for `([[` on any line ending in `]])`. Fix every hit by aliasing the date and moving it earlier in the phrase per the "Dates as contextual anchors" rule.
- **Em-dashes**: remove every `—`. Split into two bullets, nest a sub-bullet, or reword. The user does not write with em-dashes.
- **Non-natural AI language**: strip corporate/LLM tells — no "leveraged", "delved into", "streamlined", "showcased", "navigated", "fostered", "seamless", "robust", "underscored", inflated verbs, or hedging. Match the user's terse, lowercase, first-person voice.
