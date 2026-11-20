---
name: weekly-narrative
description: Generate and insert a prose narrative summary into a weekly Obsidian note. Use when the user invokes /weekly-narrative, says "summarize week of X", "add weekly narrative", "generate weekly summary for [date]", or wants an AI-written summary written into a weekly note. Trigger even if the user just says "narrative for this week" or "write up last week".
---

Generate a prose narrative summary for a given week and insert it into the weekly note's frontmatter.

Weeks run **Monday–Sunday**. The weekly note filename follows the format `Week Ending <sunday-date>.md` (e.g. week of March 15 2026 → `Week Ending 2026-03-15.md`).

## Step 1: Resolve the Week

Get today's date from the system context, or from the user's argument if provided.

Compute the Monday and Sunday bounding the target week:
- Sunday of the week = the most recent Sunday on or before the given date (if given date is Sunday, that's it)
- Monday = Sunday - 6 days

State the week before proceeding: `"Week: 2026-03-09 (Mon) → 2026-03-15 (Sun) — proceeding."` No need to wait for confirmation unless the date argument was ambiguous.

## Step 2: Check the Weekly Note

Weekly notes are at `doc/notes/periodic/weekly/`. The filename is `Week Ending <sunday-date>.md`.

Read the file if it exists. If `ai_summary` already exists in the frontmatter, ask the user before overwriting. If the file doesn't exist, you'll create it at write time.

## Step 3: Read Daily Notes

Daily notes are at `doc/notes/periodic/daily/`. Glob for the full Mon–Sun date range (e.g. `2026-03-09.md` through `2026-03-15.md`).

Read every file that exists. If fewer than 3 are found, note it briefly but proceed — partial weeks are fine.

## Step 3b: Semantic Context Discovery

Use the **query-vault** skill (read `**/query-vault/SKILL.md`). Queries: entity names that are plausibly standalone note titles in this vault — specific possessions, specific locations physically visited, named project or codebase docs. Skip event descriptions, institution names, financial/legal terms, acronyms, and anything already resolved via wikilink. Use confirmed results as additional context and as wikilink candidates.

## Step 4: Generate the Narrative

Write a 4–6 sentence narrative paragraph from the daily note content. Write it yourself — do not call another model.

Rules:
- Past tense, first person ("I", "my", "me") — never second person ("you", "your")
- Use wikilinks `[[Note Name|display]]` for projects, places, and possessions — **not people** — drawing from both resolved wikilinks and the Step 3b candidate list
- Name entities consistently — same project = same name every time
- Cover what happened, what's in motion, and anything that feels unresolved
- No bullet points, no structure — just a single prose paragraph
- Plain, natural voice — no em-dashes and no AI/corporate tells ("defined by", "navigated", "underscored", "leveraged", "delved into", "seamless", "robust")

## Step 5: Write to Frontmatter

Add `ai_summary` as a folded scalar to the weekly note's YAML frontmatter:

```yaml
ai_summary: >
  Spent most of the week... [narrative continues as wrapped lines, 2-space indented]
```

Use YAML folded scalar (`>`): indent each line of the paragraph by 2 spaces. Line-wrap at ~100 chars for readability. The `>` folds line breaks into spaces when parsed, producing a clean single paragraph.

If `ai_summary` already exists, replace it. Re-read the file immediately before editing. Edit the file in place — do not rewrite the whole file.

If the file doesn't exist, create it with only the frontmatter block containing `ai_summary`.
