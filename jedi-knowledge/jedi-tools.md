# jedi-tools

**Repository:** https://github.com/jcsda-internal/jedi-tools
**Branch tracked:** develop
**Role in JEDI:** shared utility scripts, reference data, and tooling used across the JEDI ecosystem (not part of the bundle build).

## What it is

`jedi-tools` is a sibling repo to the bundle — it lives alongside
`jedi-bundle/`, not inside it. It collects scripts and reference assets
that JEDI users and developers reach for outside of the build itself
(setup helpers, conversion utilities, bookkeeping for releases, etc).

This brief is a stub until a local checkout exists. After running the
init flow's "clone jedi-tools" step, run `/updateRepos` with the
"update knowledge base" option to flesh this out from the cloned source.

## How it fits into the bundle

It does **not** build with the bundle. It's user-facing tooling consumed
by humans or by the workflow stack (skylab/ewok). See
`jedi-knowledge/workflow.md` for how the workflow stack uses these
tools, if at all.

## Gotchas

- **Requires `git-lfs`.** Install (`brew install git-lfs` /
  `apt install git-lfs`) and run `git lfs install` once per machine
  *before* cloning. Without it, large binary assets in the repo will
  not be retrieved correctly.

## Further reading

- https://jedi-docs.jcsda.org/ — search for "jedi-tools"
- This brief will be auto-populated once the repo is cloned and
  `/updateRepos` is run with KB refresh.
