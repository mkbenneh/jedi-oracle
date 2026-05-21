---
name: jedi-updateRepos
description: Pull develop on every cloned JEDI repo, summarize the diffs to whatHasChanged.md, and optionally refresh the knowledge base in jedi-knowledge/ to match the new state.
---

# /jedi-updateRepos

Pull the latest `develop` on every cloned JEDI repo under the oracle root,
write a summary of what changed to `whatHasChanged.md`, and optionally
refresh the per-repo knowledge briefs in `jedi-knowledge/` so the oracle
stays current.

## Discovery

A "cloned repo" is any of these directories that exists and is a git
working tree:

- `jedi-bundle/` itself
- `jedi-bundle/<repo>/` for each sub-repo cloned by `/jedi-getCode`
- `jedi-docs/`
- `jedi-tools/`
- `jedi-workflow/skylab/`, `jedi-workflow/ewok/`,
  `jedi-workflow/simobs/`, `jedi-workflow/r2d2/`

Skip any that don't exist. For each, detect the configured branch:
default to `develop` unless the repo's CMakeLists entry pins a tag (e.g.
`mpas`, `gsibec`) — in that case skip the pull for that repo and note it
in the report.

## Ask the user upfront

Before doing any work, ask:

> *"Do you want me to update the knowledge base, or just tell you what
> has changed?"*

Then proceed in mode **report-only** or **report + refresh KB** based on
their answer.

## Update flow

For each cloned repo:

1. Record the current HEAD: `OLD=$(git -C <repo> rev-parse HEAD)`.
2. Pull: `git -C <repo> pull --ff-only origin <branch>`. If a fast-forward
   isn't possible (local commits, conflicts), skip the pull and record
   the reason.
3. Record the new HEAD: `NEW=$(git -C <repo> rev-parse HEAD)`.
4. If `OLD == NEW`, mark the repo "no change" and move on.
5. Otherwise capture:
   - `git -C <repo> log --oneline OLD..NEW`
   - `git -C <repo> diff --stat OLD..NEW`
   - The list of changed file paths (`git diff --name-only OLD..NEW`).

## Write `whatHasChanged.md`

Overwrite `whatHasChanged.md` at the oracle root with a fresh report. Use
this structure:

```markdown
# What has changed

_Last updated: <ISO timestamp>_

## Summary
- N repos pulled, M had changes, K skipped (reason)

## <repo-name>
**old:** <OLD>
**new:** <NEW>

### Commits
<oneline log>

### Files changed
<diff --stat>

(repeat per repo with changes)

## Skipped
<list of skipped repos with reasons (pinned tag, conflicts, missing)>
```

Keep prose short — this file is meant to be skimmed.

## If the user opted to refresh the knowledge base

For each repo with changes:

1. Re-read `jedi-knowledge/<repo>.md`.
2. Re-scan the cloned repo for anything the brief calls out: top-level
   layout, key entry points, mentioned files, public APIs touched by the
   diff.
3. Update `jedi-knowledge/<repo>.md` so it accurately reflects the new
   `develop`. Preserve the file's existing structure; rewrite sections
   that are now stale; leave sections alone if the diff didn't touch
   them.
4. If the diff includes meaningful cross-repo changes (e.g. an interface
   change in `oops` that ripples into `ufo`), update the dependent briefs
   too.
5. After all updates, also refresh `jedi-knowledge/workflow.md` if any of
   skylab / ewok / simobs / r2d2 changed.

After the refresh, list the updated knowledge files for the user so they
can review and commit them. **Do not commit on the user's behalf.** These
are tracked files and the user should review the changes.

## Reporting

End with a concise summary:

- N repos checked, M updated, K skipped
- (if KB refresh): J knowledge files updated; user should review and commit
- Path to `whatHasChanged.md` for the full report
