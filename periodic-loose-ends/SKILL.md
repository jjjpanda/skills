---
name: periodic-loose-ends
description: Audit recent periodic notes (daily, weekly, monthly) for cross-note pattern matching — surfaces items that appear once without follow-up, threads that dropped between notes, and "next period" goals that keep rolling over untouched. Not summarization; detection of disappearances and stalls.
---

You are auditing periodic notes to detect loose ends through cross-note pattern matching. The goal is not to summarize what the notes say — it's to find what the notes stopped saying.

## Step 1: Locate Notes

Find periodic note directories (daily, weekly, monthly).

Find the most recent files by modification time or date-based filenames.

Target scope in order of priority:
- **Monthly**: last 2-3 files
- **Daily**: last 21-28 files
- **Weekly**: last 4-6 files

## Step 2: Read All Notes

Read every note in scope. Do not summarize yet — extract raw content for cross-note analysis.

For weekly notes: check frontmatter for `ai_summary`. If present, use it to orient yourself before reasoning through the full note. If absent, read the full weekly note directly (do not silently write `ai_summary` frontmatter during an audit; ask first if the user wants `weekly-narrative` run).

## Step 3: Cross-Note Pattern Matching

This is the core step. For each note, identify:

**Threads** — topics, projects, people, decisions, health events, financial items, or any named thing mentioned with context or intent.

Then across all notes, classify each thread:

| Signal | What to look for |
|--------|-----------------|
| **One-and-done** | Mentioned once (often with action implied), never appears again |
| **Rolling deferral** | Listed as a goal/focus in one period, reappears in next period's plan unchanged |
| **Dropped thread** | Appeared regularly, then stopped — no resolution noted |
| **Orphaned follow-up** | A note references checking back on something; no subsequent note does |
| **Unacknowledged event** | Something significant happened in a daily note, but weeklies/monthlies never reflect on it |

Ignore: routine recurring habits with no deferred intent, pure logging of completed events with no open tail.

## Step 4: Output

Group loose ends by category. Within each category, order by actionability (clearest next action first).

```markdown
## Loose Ends — [date range audited]

### [Category: e.g. Projects / Financial / Health / Personal / Relationships]

**[Thread name]** · *[signal type]*
> First appeared: [note + date] — "[relevant quote]"
> Last appeared: [note + date] — "[relevant quote]"
> Pattern: [one sentence describing what's missing, e.g. "Mentioned wanting to call doctor after X, never appears again"]

---
```

At the end, include a short **Meta Patterns** section: are deferrals concentrated in one category? Is there a time period where threads consistently drop?

## Principles

- Quote the actual note text; don't paraphrase intent.
- Do not invent action items — only surface what the notes themselves implied but didn't resolve.
- If a thread is ambiguous (maybe resolved off-note), flag it as such rather than omitting it.
- Fewer, high-signal findings beat an exhaustive list of noise.
