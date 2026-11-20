---
name: review-full
description: Deep code review with symbol call-path tracing (LSP when available) and validated findings only
---

## Language Standard for User Output

Follow the Communication section in the global rules file (`CLAUDE.md` / `AGENTS.md` / `GEMINI.md`).

## Scope

Three modes only:
- `staged` → `git diff --cached`
- PR number (e.g. `42`) → `gh pr view 42 --json number,headRefOid,baseRefName,headRepository`, then `gh pr diff 42`
- no argument → `git diff HEAD`

**Before proceeding**, use your interactive question tool (`AskUserQuestion` / `ask_question`) to show the detected scope and confirm with the user.

If PR: run `gh repo view --json owner,name` for inline comment API calls.

---

## Review

Two passes. Run both.

### Pass 1 — Claims → code

Read the diff (and PR description if available) as a list of behavioral claims: new guards, new branches, new constants, renamed semantics, changed invariants. For each claim:

- Trace the execution path from an entry point to the new code.
- If the repo has test infrastructure (test files exist alongside or near the changed code), confirm at least one test exercises that path. If none does, it is a finding.
- Skip the test-coverage check entirely for repos with no test files, config-only changes, or non-code files.
- Also scan for removed guards, validations, or invariants — their absence is a finding if no replacement exists.

**Doc gap check** — run only when the diff includes documentation files (READMEs, guides, `.md`/`.rst`/`.txt` docs):

For each doc section that describes a script or command, read the full script (not just changed lines) and extract its complete behavioral surface:
- Permissions/elevation required (UAC, sudo, admin rights)
- Env vars the script reads or requires
- Side effects users must know about (indefinitely-running processes, broad process kills beyond stated scope, files written outside the working directory, network calls)
- Implicit prerequisites (tools, config files, env vars) — especially ones the old doc mentioned that the new doc dropped

Flag any of these that the doc section omits or understates. A missing claim in the doc is a finding just like a false claim.

### Pass 2 — Code → failures

Skip Pass 2 for non-code files (`.md`, `.txt`, `.json`, `.yaml`, config files, etc.).

When an `LSP` tool is available in the active harness (in Claude Code, it is deferred — run `ToolSearch(query: "select:LSP")` once before the first call so `documentSymbol`/`findReferences`/etc. resolve), use LSP-driven tracing; otherwise enumerate symbols from the full file and trace callers, references, definitions, and types using workspace text search and full-file reads.

For each changed file (smallest → largest):

1. Read the full file.
2. Enumerate all symbols in the file (`documentSymbol` via LSP when available, or by inspecting declarations in the file).
3. For each changed symbol, trace fully before forming any claim:
   - If LSP is available:
     - `prepareCallHierarchy` + `incomingCalls`/`outgoingCalls` to map the call graph
     - `findReferences` to see every usage site before claiming misuse
     - `goToDefinition`/`goToImplementation` to navigate dependencies
     - `hover` to verify type signatures and API contracts
     - Fall back to workspace text search only when LSP cannot resolve a symbol
   - If LSP is unavailable:
     - Search across the workspace for symbol usages, callers, and implementations, and read each definition and call site in full
4. For every candidate issue that survives tracing:
   - Read every file involved in the call path.
   - Trace callers (`incomingCalls` or search-based call-chain reads) up the full caller chain to a user-visible entry point. Code only reachable from diagnostics, stats collection, or logging paths does not qualify — discard it.
   - Confirm a concrete, reproducible failure: specific inputs → specific bad output, reachable from a relevant entry point with user-visible impact. Structural asymmetry alone does not satisfy this bar.
   - Verify the issue isn't already handled elsewhere via `findReferences` or workspace text search.
   - If LSP cannot resolve a symbol and text search is the fallback (SQL, CUDA, external schemas, or non-LSP harnesses): do not assert absence of an external artifact (schema column, table field, enum value) based on pattern matching. If it cannot be verified by reading actual definitions in the repo, discard it.
   - If you cannot state exact conditions that trigger it, discard it.
5. Only report findings that survive full validated tracing.

---

## Output

Flat list, confirmed findings only:

```
## Review: <scope description>

- file/path.ts:42 — description
- file/path.ts:88 — description
```

No severity tiers. No prose. No positive feedback. If nothing found: `No findings.`

**GH inline comments** (PR only) — one call per finding:

```bash
gh api repos/{owner}/{repo}/pulls/{pr_number}/comments \
  -f body="REVIEW AGENT: <summary>

<Cross-file evidence — 2-3 sentences max. Omit if single-file.>" \
  -f commit_id="{headRefOid}" \
  -f path="{file/path}" \
  -F line={line_number} \
  -f side="RIGHT"
```

**Fallback** — `side=RIGHT` only anchors to lines present in the new file, so a finding on a removed guard or on context outside a hunk returns 422. On 422, retry once as a file-level comment: drop `line` and `side`, add `subject_type`, and carry the line number in the body since the anchor no longer holds it.

```bash
gh api repos/{owner}/{repo}/pulls/{pr_number}/comments \
  -f body="REVIEW AGENT: <summary> (line {line_number})

<Cross-file evidence — 2-3 sentences max. Omit if single-file.>" \
  -f commit_id="{headRefOid}" \
  -f path="{file/path}" \
  -f subject_type="file"
```

Both forms live on `pulls/{pr}/comments`, so either one is fetched by the same listing and deleted by `gh api -X DELETE repos/{owner}/{repo}/pulls/comments/{id}`. If the fallback also fails, report the finding as unposted — never drop it silently.
