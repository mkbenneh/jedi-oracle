---
name: jedi-investigateIssue
description: Search GitHub issues the user is involved in (across all repos) by keyword, let the user pick one, then pull the full issue with the gh CLI and produce a research report covering the problem and possible solutions.
---

# /jedi-investigateIssue

Help the user dig into a GitHub issue they care about — without needing to
know which repo it lives in. The skill searches across all repos the user
has access to (default scope: `--involves=@me`), surfaces matches against a
user-supplied keyword, lets the user pick one, and then pulls the issue and
its discussion with the `gh` CLI so the oracle can research it and write a
report with possible solutions.

This skill assumes the `gh` CLI is installed and authenticated. If `gh auth
status` shows the user is not logged in, ask them to run `gh auth login`
first. For private repos behind SSO, mention `gh auth refresh` may be
needed if results look incomplete.

## Flow

1. **Ask for search terms.** Prompt the user for the keyword(s) they want
   to filter on. Examples: *"CADRE HelpDesk"*, *"ufo bias correction"*,
   *"crash on amip"*. Multiple terms are OR'd by default in GitHub search;
   suggest quoting a phrase if they want it exact.

2. **Confirm scope.** Default is `--involves=@me` (issues where the user
   is author, assignee, mentioned, or commented). Mention the alternates
   in case they want to narrow:
   - `--assignee=@me` — only issues assigned to them
   - `--mentions=@me` — only issues that @-mention them
   - `--author=@me` — only issues they opened

   Also ask whether to include closed issues (default: open only) and
   whether to scope to a specific org (e.g. `--owner=jcsda-internal`).

3. **Run the search.** Use `gh search issues` with a JSON projection so
   the output is parseable. Example:

   ```bash
   gh search issues --involves=@me --state=open \
     --json number,title,repository,url,state,updatedAt,labels,assignees \
     --limit 30 \
     'CADRE HelpDesk'
   ```

   Add `--owner=<org>` if the user scoped to one. If the user picked a
   different relationship flag, swap `--involves` for that.

4. **Display the matches as a numbered list.** For each hit show:
   `<n>. <owner/repo>#<number> — <title>  (<state>, updated <date>)`
   Include labels in parentheses if any look meaningful (e.g. `CADRE`,
   `HelpDesk`, `bug`). Cap the visible list at ~20; if there are more,
   say how many were truncated and offer to refine the search.

   If there are zero matches, suggest broadening: drop `--state=open`,
   loosen the keyword, or switch from `--assignee` to `--involves`.

5. **Ask which issue to investigate.** Accept a number, an
   `owner/repo#number` reference, or a URL. If the user is unsure, offer
   to print a one-line preview for any specific entry before they commit.

6. **Pull the full issue.** Once chosen, fetch it with comments:

   ```bash
   gh issue view <number> --repo <owner/repo> --comments \
     --json number,title,state,author,assignees,labels,body,comments,url,createdAt,updatedAt,closedAt,milestone
   ```

   Also list any linked PRs / cross-references:

   ```bash
   gh issue view <number> --repo <owner/repo> \
     --json projectItems,closedByPullRequestsReferences 2>/dev/null || true
   ```

   (Some fields vary by `gh` version — if a field is rejected, drop it
   and retry; don't block the investigation on this.)

7. **Research.** Use everything available to understand the issue:
   - **Knowledge base** — read the relevant `jedi-knowledge/<repo>.md`
     brief for the repo the issue lives in, plus `jedi-knowledge/jedi-tips.md`
     and `jedi-knowledge/coding-practices.md` per CLAUDE.md guidance.
   - **Cloned source** — if the issue mentions files, functions, error
     messages, or stack traces, grep the cloned repos in the workspace
     (`jedi-bundle/`, `jedi-docs/`, `jedi-tools/`, `jedi-workflow/`,
     relative to the workspace root per CLAUDE.md — normally `../`)
     for the relevant symbols. Read the actual code, don't just trust
     the brief.
   - **Linked PRs / commits** — `gh pr view <num> --repo <owner/repo>`
     for any referenced PRs; `git log` / `git show` in the relevant
     clone for referenced commits.
   - **External knowledge** — check `externalKnowledge.md` if present
     for resources the user has previously added that might apply.

8. **Produce the report.** Write a structured report directly in the
   conversation (don't create a file unless the user asks). Suggested
   sections:

   ```
   ## Issue summary
   <owner/repo#number — title — state — assignees>
   <1–3 sentence problem statement in plain language>

   ## Context
   <what subsystem this touches, who reported it, when, related issues/PRs>

   ## What the discussion says
   <key points from the body and comments — quote sparingly, cite commenters>

   ## What the code shows
   <findings from grepping/reading the cloned source, with path:line cites>

   ## Possible solutions
   <ranked list of candidate fixes or next investigative steps, with
    tradeoffs. Be honest when the right path is unclear.>

   ## Open questions
   <things the oracle couldn't resolve from available info — what to
    ask the reporter, what to test, what data to gather>

   ## References
   <issue URL, linked PRs, knowledge files consulted, source files cited>
   ```

   Keep cites concrete: `jedi-knowledge/ufo.md`, `ufo/src/ufo/Foo.cc:142`,
   PR URLs. Don't invent file paths or function names — verify before
   citing.

9. **Offer follow-ups.** At the end, ask whether the user wants to:
   - Save the report to a file (e.g. `investigations/<repo>-<num>.md`)
   - Save a project memory so future sessions remember this investigation
     (see the memory system referenced by `/jedi-getProject`)
   - Draft a reply comment, a PR plan, or a reproducer

## Notes

- **Don't auto-execute fixes.** This skill produces a report and
  recommendations; any code change should go through a separate plan
  with the user.
- **Honor `gh` auth scope.** `gh search` only returns repos the user's
  token can see. If results seem light, suggest `gh auth status` and
  `gh auth refresh` (for SSO orgs).
- **Respect CLAUDE.md operating principles.** Cite knowledge files,
  cite code with `path:line`, trust current code over cached briefs,
  and be honest about uncertainty. If a brief contradicts the cloned
  source, offer `/jedi-updateRepos` to refresh.
- **Keep it scoped.** If the issue is sprawling (long thread, many
  linked PRs), ask the user which angle to dig into first rather than
  trying to cover everything at once.
