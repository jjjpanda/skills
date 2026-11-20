---
name: expand-daily-note
description: Take a sparse, shorthand daily note and expand it into the user's natural writing voice — without editorializing, assuming, or adding filler. Use when the user wants to flesh out, expand, or elaborate a terse daily note.
---

Expand a sparse daily note into the user's natural writing voice.

## Step 1: Read the Target Note

The user must provide the full path to the note. Read it in full. Note:
- Frontmatter structure and fields
- Body structure (headings, dividers, sections)
- What's abbreviated, vague, or just a keyword

If no path was provided, ask for it before proceeding.

## Step 2a: Locate and Read Recent Daily Notes for Style Context

List `*.md` files in the same directory as the target note sorted by modification time. Read the 4–6 most recent notes (excluding the target).

For each of these style-context notes, extract any wikilinks (`[[...]]`) and resolve them to their actual files — read any that are non-periodic notes (i.e. skip daily/weekly/monthly wikilinks here). This surfaces richer prose from project notes, location notes, and item notes that the user has actually written, giving you a larger sample of their voice.

From all of this material, internalize:

- **Tone**: formal vs. casual, sentence case vs. lowercase, dry vs. expressive
- **Sentence structure**: full sentences vs. fragments vs. declarative bursts
- **Wikilink conventions**: what kinds of things get wikilinked (places, projects, people, none?)
- **Level of detail**: how many words per beat, whether elaboration ever appears

If no other notes exist in the directory, skip this step and note that you'll infer voice from the target note alone.

## Step 2b: Resolve Wikilinks for Context

For every wikilink in the target note, resolve it to its actual note file and read it. This is critical — wikilinks are often the only signal of what an entry means, and guessing from the display text is unreliable.

- **Locations**: read the location note for place type and context
- **Projects/code**: read the project note to understand what it actually is (e.g. don't conflate separate projects)
- **Possessions/items**: read the item note for context on what it is and why it's mentioned
- **People**: people are not notes — skip
- **Periodic notes** (daily/weekly wikilinks): read them for referenced context if relevant

If a wikilink uses an alias (`[[actual note|display text]]`), navigate to the actual note, not the display text. If a wikilink target can't be found by exact name, use file or text search to locate it.

Do not proceed to Step 2c until all meaningful wikilinks are resolved.

## Step 2c: Semantic Backlink Discovery

Use the **query-vault** skill (read `**/query-vault/SKILL.md`). Queries: entity names extracted from each line (not whole lines) — specific possessions, specific locations physically visited, named project or codebase docs. Skip event descriptions, institution names, financial/legal terms, acronyms, and anything already resolved via wikilinks in Step 2b. Use confirmed results as the candidate backlink list for Step 6.

## Step 3: Identify Gaps and Ambiguities

Cross-reference the sparse note against the style patterns. Flag:
- Entries too vague to expand without context (single words, abbreviations, acronyms)
- Events that could mean multiple things (e.g. "school" — drop off? pick up? attended?)
- Anything emotionally charged or unusual that warrants explanation
- Things that appear disconnected or out of sequence

## Step 4: Ask Targeted Questions (Max 2 Rounds)

Use your interactive question tool (`AskUserQuestion` / `ask_question`) — not freeform text — so answers are captured cleanly.

**Round 1**: Ask about the main ambiguous beats. Group related questions. Provide options where reasonable but always allow freeform answers. Keep questions grounded in the actual note content.

**Round 2** (if needed): Follow up on remaining gaps surfaced after Round 1 answers.

Do not ask about anything the user confirmed is unimportant or said to omit.

## Step 5: Interpret Answers Faithfully

- "normal, nothing notable" → one plain line
- "not important" or "skip it" → omit entirely
- Match the user's word choices and investment level per topic
- Never embellish, infer emotion, or add interpretation beyond what was confirmed

## Step 6: Draft the Expanded Body

Write in the user's voice as inferred from Step 2. Present the full draft to the user before writing anything. Apply wikilink conventions observed in recent notes. Use the candidate backlink list from Step 2c to add `[[wikilinks]]` where the linked note clearly adds context — don't force-link everything. Do not add transition phrases, commentary, or filler sentences.

State clearly: "Here's the draft — let me know if anything needs adjusting before I write it."

Wait for approval or corrections.

## Step 7: Write the File

Read the file again immediately before editing (required). Edit the file in place to replace only the body content. Do not touch frontmatter. Preserve existing structure (headings, dividers, section order).
