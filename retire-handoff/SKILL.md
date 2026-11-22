---
name: retire-handoff
description: Prepare for session end. Hand off context to GitHub issue/PR/comment or markdown file, then give a prompt for the next session to pick back up. Use only when the user asks to end the session, e.g. "wrap up the session", "prepare for retirement", "handoff to next session".
---

Follow the Communication section in the global rules file (`CLAUDE.md` / `AGENTS.md` / `GEMINI.md`).

1. **Wrap up.** Let running jobs and subagents finish naturally; note any still running and any uncommitted work.
2. **Hand off** to an existing GitHub issue/PR comment, else a uniquely named markdown file. If the project allows, make the file temporary and untracked. Write for a reader with no context: goal, state, next steps, decisions, learnings — with links and SHAs. No secrets.
3. **Reply** with what got done and a short next-session prompt in a code block. The prompt gives the handoff link, says to verify current state first, and, if the file is temporary, to delete it once work has resumed.
