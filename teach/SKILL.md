---
name: teach
description: Teach a technical concept piece by piece — calibrate to the user's level, build from what they know, check understanding at each step, and follow their curiosity rather than a fixed outline
---

## Language Standard for User Output

Follow the Communication section in the global rules file (`CLAUDE.md` / `AGENTS.md` / `GEMINI.md`).

## When to use

The user wants to *understand* something interactively, piece by piece — not just get a one-shot explanation, fact, or fix. They may say "walk me through", "teach me", or invoke `/teach <topic>`. (For a one-shot explanation of a mechanism, use `explain`.)

## Process

### 1. Calibrate

Before explaining anything, find two things with your interactive question tool (`AskUserQuestion` / `ask_question`):

**Level** — where the user sits on the novice-to-expert spectrum *for this topic*. Do not assume from role. Someone who wrote the code may not follow the math; a statistician may not follow the code. Offer 3 concrete levels, not generic labels.

**Focus** — which part is unclear. Offer 2-3 specific sub-topics plus "all of the above". This prevents a lecture on parts they already know.

### 2. Build piece by piece

Break the topic into a chain of self-contained pieces, ordered by dependency: each piece uses only concepts from earlier pieces.

Per piece:
- **One heading**, named for the concept ("From trades to cells", not "Step 1")
- **One concrete example** with real numbers. Not "consider a value X" — use "47 buys out of 83 trades"
- **One analogy max**, and only when the real mechanism is hard to picture. The analogy must map precisely — if it leaks, cut it. Prefer showing the real thing
- **No jargon without a definition** on first use. Define inline, not in a glossary
- Keep each piece to 4-8 sentences of body text plus the example

### 3. Check understanding after each piece

Use your interactive question tool (`AskUserQuestion` / `ask_question`) with 2-3 options:
- One option = "got it, move on"
- One option = "go deeper on [specific sub-topic]"
- One option = a common confusion you anticipate

**Read the user's answer carefully.** They may type a free-form question instead of picking an option. If they do:
- Answer the question concisely (3-5 sentences)
- Then return to the main thread — don't let a tangent derail the sequence

### 4. Follow their curiosity

If the user's question reveals a different gap than expected, pivot. The sequence serves the user, not the outline. Signs to pivot:
- They ask "but why not just X?" — they see a simpler path you haven't addressed
- They say "wait, go back" — a prior piece didn't land
- They challenge a premise — take it seriously, they may be right

### 5. Synthesize

After the last piece, give a 3-4 sentence summary that connects all pieces into one coherent picture. No new concepts here — just the thread that ties them together.

## Rules

- **Never talk down.** The user built the system. They're asking because the domain crosses a boundary they don't specialize in.
- **Never dump.** No piece should have more than one new concept. If you're writing "furthermore" or "additionally", you're combining two pieces.
- **Never assume the question behind the question.** If they ask about X, answer X. Don't preemptively explain Y because you think they'll need it — wait until they ask or until piece order demands it.
- **Examples beat abstractions.** "B ~ BetaBin(N, p₀, κ)" means nothing until you say "like flipping a slightly wobbly coin 83 times where the wobble means the buy rate drifts naturally between 50-56%."
- **Tangents get 3-5 sentences, then return.** Acknowledge the tangent, answer it, explicitly resume: "Back to the main thread —"
- **Tables for comparisons.** If two things differ, show a table. Do not describe the differences in prose.
- **No "let me explain" preamble.** Start with the content.

## Arguments

- `/teach <topic>` — teach the named topic using this process
- `/teach` (no arg) — ask the user what they want to understand

## Anti-patterns

- Writing a 500-word block before asking calibration questions (`AskUserQuestion` / `ask_question`). Calibrate first.
- Asking "does that make sense?" without specific options. Give them something concrete to react to.
- Explaining all edge cases before the user understands the happy path.
- Using an analogy that requires its own explanation.
- Answering a tangent with another tangent.
