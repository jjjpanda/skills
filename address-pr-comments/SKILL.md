---
name: address-pr-comments
description: Read PR review comments, validate each against code with symbol tracing (LSP when available), then apply fixes for confirmed valid issues
---

## Language Standard for User Output

Follow the Communication section in the global rules file (`CLAUDE.md` / `AGENTS.md` / `GEMINI.md`).

## Scope

Requires a PR number argument (e.g. `42`).

Run `gh repo view --json owner,name` to populate `{owner}` and `{repo}`, then run in parallel:
- `gh pr view {pr} --json number,headRefOid,baseRefName,headRepository,title`
- `gh pr diff {pr}`
- `gh api --paginate repos/{owner}/{repo}/pulls/{pr}/comments` — inline review comments
- `gh api --paginate repos/{owner}/{repo}/pulls/{pr}/reviews` — review-level comments
- `gh api --paginate repos/{owner}/{repo}/issues/{pr}/comments` — general PR comments

`--paginate` on all three: each returns 30 per page by default, so the newest comments are missing without it.

**Branch check**: run `git rev-parse HEAD` and compare against `headRefOid` from the PR metadata. If they differ, stop and tell the user which branch/commit is checked out vs. what the PR expects — do not proceed until the working tree matches the PR source branch.

**Before proceeding**, use your interactive question tool (`AskUserQuestion` / `ask_question`) to confirm the PR title, number, and total comment count with the user.

---

## Pre-Validation Discussion

Before any code tracing, for each comment extract the claimed issue and your interpretation of it, then present a summary table:

```
| # | Author | File:Line | Claim (your read) |
|---|--------|-----------|-------------------|
| 1 | alice  | foo.ts:42 | null check missing before .value access |
...
```

Use your interactive question tool (`AskUserQuestion` / `ask_question`): "Does this interpretation look right? Flag any misreads before I start tracing."

Incorporate any corrections, then proceed.

---

## Validation

When an `LSP` tool is available in the active harness (in Claude Code, it is deferred — run `ToolSearch(query: "select:LSP")` once before the first trace so `documentSymbol`/`findReferences`/etc. resolve), use LSP-driven tracing; otherwise enumerate symbols from the full file and trace callers, references, definitions, and types using workspace text search and full-file reads.

For each confirmed comment (inline, review, general):

**Explain first**: state your understanding of the claim and what a concrete failure would look like before tracing.

1. Read the full referenced file.
2. Locate the relevant symbol (`documentSymbol` via LSP when available, or by inspecting declarations in the file).
3. Trace before accepting or rejecting the claim:
   - If LSP is available:
     - `prepareCallHierarchy` + `incomingCalls`/`outgoingCalls` to verify call context
     - `findReferences` to check all usage sites
     - `goToDefinition`/`goToImplementation` to inspect dependencies
     - `hover` to confirm types and signatures
     - Fall back to workspace text search only if LSP cannot resolve the symbol
   - If LSP is unavailable:
     - Search across the workspace for symbol usages, callers, and implementations, and read each definition and call site in full
4. Trace callers (`incomingCalls` or search-based call-chain reads) up the full caller chain to a user-visible entry point. Code only reachable from diagnostics, stats collection, or logging paths does not qualify — reject it.
5. Accept the claim only if you can confirm a concrete, reproducible failure: specific inputs → specific bad output, reachable from a relevant entry point with user-visible impact. Structural asymmetry alone does not satisfy this bar.
6. If LSP cannot resolve a symbol and text search is the fallback (SQL, CUDA, external schemas, or non-LSP harnesses): do not assert absence of an external artifact (schema column, table field, enum value) based on pattern matching. If it cannot be verified by reading actual definitions in the repo, reject it.
7. Reject the claim if it's already handled elsewhere, if the concern is stylistic only, or if you cannot state exact conditions that trigger it.

---

## Fixes

For each accepted claim, apply the minimal correct fix:

- Edit only the code directly implicated — no opportunistic cleanup.
- Do not add comments.
- After edits, re-trace (with LSP when available, otherwise workspace search + full-file reads) to confirm the fix doesn't break call sites or type contracts.

---

## Output

```
## PR #{pr}: {title}

### Accepted & Fixed
- comment #{id} ({author}) file/path.ts:{line} — <what was wrong> → <what was changed>

### Rejected
- comment #{id} ({author}) — <one-line reason: already handled / not reproducible / stylistic>

### Skipped (no code reference)
- comment #{id} ({author}) — <brief note>
```

No prose. No positive feedback. If no comments: `No review comments found.`

---

## Cleanup

Use your interactive question tool (`AskUserQuestion` / `ask_question`) to ask: "Ready to delete the addressed comments?"

If confirmed, delete each accepted (fixed) comment by its source type:
- Inline review comment: `gh api -X DELETE repos/{owner}/{repo}/pulls/comments/{id}`
- General PR comment: `gh api -X DELETE repos/{owner}/{repo}/issues/comments/{id}`

Do not delete rejected or skipped comments.
