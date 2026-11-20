---
name: pr-review-loop
description: Drive a PR to a clean review — isolated review and fix agents alternate each round, committing and pushing, until no findings remain. Use when asked to keep reviewing and fixing a PR until it passes.
---

## Language Standard for User Output

Follow the Communication section in the global rules file (`CLAUDE.md` / `AGENTS.md` / `GEMINI.md`).

## Scope

Requires a PR number argument.

- `gh repo view --json owner,name`
- `gh pr view {pr} --json number,headRefOid,baseRefName,headRepository,title`

Step 0 runs the HEAD-vs-`headRefOid` check every round, round 1 included, so there is no separate setup branch check.

**Before proceeding**, one call to your interactive question tool (`AskUserQuestion` / `ask_question`) covering all of:
- PR number + title, for confirmation
- Iteration cap (default 3)
- Authorization to auto-commit, auto-push, and delete review comments every round for the rest of the loop

A yes covers every remaining round. It does not remove harness permission prompts — `git add/commit/push` and `gh api` are not allowlisted by default, so each round still prompts for the commit, the push, and every comment call. Allowlist those yourself for a hands-off run.

---

## Decision ledger

This session (the orchestrator) keeps an in-memory ledger across rounds. Per finding: file path, the claim in one normalized phrase, and the verdict (accepted / rejected / skipped). Per round: round #, and the commit hash if one was made.

Key every finding on file path + claim, never on file:line. Each round's commit shifts line numbers, so a line-keyed ledger stops matching its own earlier entries after the first fix lands.

Never pass the ledger to either agent, and never pass the review agent's trace to the fix agent. Each round both agents start cold: the review agent sees only the current diff, the fix agent sees only the current diff and the posted review comments.

The fix agent does read comments the review agent wrote, so that much reasoning crosses over by design. What the separation actually buys is isolation between rounds — no agent inherits an earlier round's conclusions, so every round re-derives its position from the code. Only the orchestrator sees the full history.

---

## Loop

Repeat until a stop condition hits:

0. **Refresh head** — re-run `gh pr view {pr} --json headRefOid`, `git rev-parse HEAD`, and `git symbolic-ref -q HEAD`. The two SHAs must match, and `symbolic-ref` must resolve to a branch. Stop and report if either check fails.

   Round 1's SHA goes stale the moment step 7 pushes, and inline comments posted against a stale `commit_id` fail or anchor to the wrong commit. A SHA mismatch means the last push never landed, or someone else pushed to the branch. A detached HEAD passes the SHA check and then fails at step 7 with no upstream — after a full round of agent work has already run. Either way, stop — never review one tree and comment on another. Pass the refreshed value to the review agent.

1. **Review agent** — spawn a fresh isolated subagent with no memory of prior rounds:
   Use the harness's subagent tool, run in the foreground with the default model. Instruct it to follow the `review-full` skill with the PR number.
   Give it only: PR number, repo, current `headRefOid`, current diff. Instruct it to skip every interactive confirmation gate (`AskUserQuestion` / `ask_question`) in that skill — nobody is there to answer them. It posts inline GH comments per finding and reports back a flat finding list, or `No findings.`

   Tell it to flag any finding whose comment POST failed. `review-full` retries a 422 as a file-level comment, so this should be rare — but an unposted finding reaches no one downstream, because every later step finds comments by their `REVIEW AGENT:` body prefix.

   Record each unposted finding in the ledger as Skipped before step 5. The fix agent never sees it, so it returns no verdict for it, and without a ledger entry step 3 has nothing to match — the next round's review agent re-raises it, fails to post it again, and the loop runs to the cap with no commit. List these in the output.

2. **Stop check** — if the review agent reports `No findings.`, exit the loop. Success.

3. **Repeat-verdict filter** — the orchestrator (never an agent) checks each new finding against the ledger, matching on file path + claim, not file:line. Earlier rounds' commits have moved the lines; a line match would miss the repeat and let it through. If a finding repeats one already marked Rejected or Skipped, delete its comment and drop it — don't send it to the fix agent.

   Skipped counts the same as Rejected here. Neither verdict produces a fix, so neither changes the diff, so the next round's review agent raises the identical finding again. Filtering only on Rejected leaves Skipped findings cycling to the round cap with no commit and no progress.

   ```bash
   gh api --paginate repos/{owner}/{repo}/pulls/{pr}/comments \
     --jq '.[] | select(.body | startswith("REVIEW AGENT:")) | {id, path, body}'
   ```

   Match the comment to drop by path + claim text, then `gh api -X DELETE repos/{owner}/{repo}/pulls/comments/{id}`. `--paginate` is required: the endpoint returns 30 per page sorted oldest-first, so on a PR with existing discussion this round's comments sit past page 1.

   This is orchestrator-only bookkeeping; it doesn't break the wall between agents. If every finding this round gets dropped this way, exit the loop and report `Re-raised only findings already rejected or skipped` — not `No findings.`, which would claim a clean review the code hasn't earned.

4. **Flip-flop check** — compare what's left against the ledger. If any finding contradicts a fix already applied in an earlier round (same file/area, opposite direction), halt before touching it. Use your interactive question tool (`AskUserQuestion` / `ask_question`) to show the user both the earlier accepted fix and the new contradicting finding, and get a ruling. Apply the ruling by editing or deleting that comment yourself before the fix agent runs — locate it by path + claim with the step 3 fetch — so the fix agent only ever sees an already-resolved comment.

5. **Fix agent** — spawn a second fresh isolated subagent, walled off from the review agent's trace:
   Use the harness's subagent tool, run in the foreground with the default model. Instruct it to follow the `address-pr-comments` skill with the PR number.
   Give it only the PR number and the current diff. Instruct it to skip every interactive confirmation gate (`AskUserQuestion` / `ask_question`) in that skill, and to act only on comments whose body starts with `REVIEW AGENT:`.

   That skill's own scope step fetches all three comment endpoints, so the fix agent will read human comments no matter what you hand it. The prefix restriction is the only thing keeping it from acting on them — give it as a hard constraint, not a preference. An agent running with its confirmation gates stripped must never be the thing deciding whether a human's comment is in scope.

   Steps 3 and 6 find comments by the same prefix, so the orchestrator never has to collect IDs up front or match findings to comments by line. The fix agent reports Accepted/Rejected/Skipped with the comment IDs it read; that is where the ledger's verdicts come from. Tell it to skip that skill's Cleanup step and delete nothing — step 6 does all deletion.

6. **Sweep** — append this round's outcome to the ledger, then delete every `REVIEW AGENT:` comment on the PR, whatever its verdict:

   ```bash
   gh api --paginate repos/{owner}/{repo}/pulls/{pr}/comments \
     --jq '.[] | select(.body | startswith("REVIEW AGENT:")) | .id'
   ```

   Then `gh api -X DELETE repos/{owner}/{repo}/pulls/comments/{id}` for each. The ledger keeps the verdicts; the PR keeps nothing. Any comment left behind is still there next round for a fresh fix agent to re-argue from scratch — and it may accept what the last round rejected. Deleting is safe because the ledger still lets step 3 catch the finding if the review agent rediscovers it.

7. **Commit and push** — if the fix agent made no changes this round (everything rejected or skipped), skip this step. Otherwise: one commit, single-line message, then push. Already authorized in the setup gate — don't ask again.

   If the push is rejected (non-fast-forward, branch protection), stop the loop and report it. Never retry, never force. The commit stays local, so the next round would review a tree the PR head doesn't have, and every comment it posts would anchor to the wrong commit.

8. **Round cap** — if the round count reaches the cap and the review hasn't cleared, stop. Report the ledger instead of starting another round.

9. Otherwise, go to step 0.

---

## Output

```
## PR #{pr} loop: {N} round(s)

- Round 1: {count} findings → {accepted}/{rejected}/{skipped} → commit {sha | no commit}
- Round 2: ...

Result: {one of}
  No findings.
  Re-raised only findings already rejected or skipped — {count} still open.
  Capped at {N} rounds.
  Halted — engineering conflict at file:line.
  Stopped — push rejected at round {N}.
  Stopped — local HEAD does not match PR head.
  Stopped — HEAD is detached, no branch to push.
```

No prose beyond this. If halted on a conflict, show the two contradicting findings side by side. If any finding's comment failed to post, list those findings under the rounds — they never reached a fix agent.
