---
name: explain
description: Explain how a mechanism or design works by abstracting it into plain, simple terms — skipping line-by-line code traces, buzzwords, and boilerplate. Use whenever the answer is a mechanism, mental model, or comparison.
---

## Language Standard for User Output

Follow the Communication section in the global rules file (`CLAUDE.md` / `AGENTS.md` / `GEMINI.md`).

Abstract the mechanism before referencing code. Give the reader a clear mental model in plain English, not a compressed transcript of lines or a rigid template.

## Rules

- **Lead with the core idea in plain English.** State what the system guarantees, transforms, or prevents in 1–2 short sentences. If the sentence relies on internal function names or config keys to make sense, simplify further (a quiet parenthetical code anchor at the end is fine).
- **Group by purpose, not code order.** Collapse pass-through calls, wrappers, and boilerplate into 2–4 meaningful parts. Leave out details that do not change the outcome.
- **Use short, normal sentences.** Plain grammar with `if`, `because`, and `so` is clearer than telegraphic fragments or arrow chains. Do not stack nouns together just to save a few tokens.
- **Pick the simplest format that fits:**
  - **Short numbered steps or bullets** for a sequence or a small set of rules.
  - **A small indented tree** only when branching outcomes are the main point.
  - **A table** only when comparing 2+ options across shared criteria (skip rows where they are identical) or showing what breaks at each step.
- **Keep code anchors quiet.** When explaining local code, put `(file.ext:line)` or symbol names in parentheses at the end of a sentence or bullet so the reader can locate the source without wading through identifiers. Skip code anchors for general concepts.
- **Add caveats only when they matter.** Mention a trap, contrast, or fix only if it would genuinely surprise a caller or cause a bug. Never force a contrast (`X, not Y`) or an insight footer.

## Examples

### Code mechanism

The database pool caps active connections at 10 and makes extra callers wait up to 5 seconds before failing (`db/pool.js:88`, `acquire`).

1. If an idle connection is free, hand it out immediately.
2. Otherwise, open a new connection until the pool reaches 10.
3. At 10 active connections, queue the caller on a 5-second timer; if nothing frees up in time, fail with `PoolExhausted` without retrying.

If a single slow query holds a slot long enough to fill the queue, every waiting caller times out together after 5 seconds.

### Failure across steps

Deleting an upload removes the files from disk before removing the database record, so a mid-request disconnect leaves orphaned UI state (`routes/uploads.py:44`, `DELETE /uploads/:id`).

| Step | Action | If client disconnects after step 1 |
| --- | --- | --- |
| 1 | Delete chunks from disk | Files are already gone (no rollback) |
| 2 | Delete DB row | Skipped, so the UI still lists a broken file |
| 3 | Purge CDN cache | Skipped, so CDN serves stale hits until TTL, then 404 |

Marking the DB row as deleting first and cleaning up disk and CDN asynchronously avoids broken UI entries.

### Conceptual comparison

Optimistic and pessimistic locking both prevent conflicting edits, but they differ on whether conflicts are rare or common:

| Axis | Pessimistic locking | Optimistic locking |
| --- | --- | --- |
| Mechanism | Lock the row on read; block other writers | Read freely; check a version number on write |
| On conflict | Other callers wait | Late writer fails and must retry |
| Best when | Contention is high or retries are expensive | Reads outnumber writes and conflicts are rare |

## Avoid

- **Line-by-line code translation.** Summarizing every `if/else` or helper call in order instead of stating the higher-level behavior.
- **Forced rhetorical tics.** Avoid canned `"It is X, not Y"` contrasts, emoji tags (`≈`, `✅`, `❌`, `⚠️`, `💡`), bolded insight prefixes, or dramatic conclusions when a plain sentence works.
- **Telegraphic fragments.** Do not strip articles and conjunctions into dense chains like `stale cache → miss → silent 401`.
- **LLM buzzwords.** Avoid *under the hood*, *at its core*, *orchestrates*, *seamless*, *robust*, *first-class*, *crux*, *hinge*, and *emergent*.
- **Saying it twice.** Do not state the same point in an opener, a table, and a footer.
