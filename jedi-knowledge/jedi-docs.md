# jedi-docs

**Repository:** https://github.com/jcsda-internal/jedi-docs
**Branch tracked by bundle:** develop
**Role in JEDI:** the source for the rendered JEDI documentation site at https://jedi-docs.jcsda.org/. Sphinx-based; not part of the bundle build.

## What it is

`jedi-docs` is the documentation source repo. It is **not** built by
the bundle — it's a standalone Sphinx project (well, two Sphinx projects
plus a directory of how-to markdown). The published site at
https://jedi-docs.jcsda.org/ is the rendered output.

It is included alongside the bundle so users have local access to the
docs and can reference / search them while working with the code.
The init flow in CLAUDE.md offers to clone it into `./jedi-docs/`.

## How it fits into the bundle

- **Not built.** The bundle does not include `jedi-docs` in its
  `ecbuild_bundle( … )` declarations. It's a sibling resource cloned
  by the oracle's init flow on user request.
- **Consumed by:** humans (and the oracle, which can read the source
  to answer questions).

## Key directories

- `docs/` — the main Sphinx project for the public JEDI
  documentation site (https://jedi-docs.jcsda.org/).
- `jedi-edu/` — a separate Sphinx project for educational / tutorial
  content.
- `howto/` — markdown-format how-to notes that live outside Sphinx.

Each Sphinx project (`docs/` and `jedi-edu/`) typically has its own
`conf.py`, source tree under `source/` (or similar), and produces an
independent rendered site.

## Common tasks

- **Preview the docs locally** — from `docs/` (or `jedi-edu/`):
  `make html`, then open `_build/html/index.html`. (Or `sphinx-build`
  invoked directly.) Sphinx + the `requirements.txt` packages must be
  installed.
- **Find a specific topic** — `grep -r <term> docs/ jedi-edu/ howto/`
  is fast. The published site has search; locally Sphinx generates
  one too.
- **Add a new page** — drop an `.rst` (or `.md` if myst-parser is
  configured) under `docs/source/<section>/`, link it from the
  section's `index.rst`. Use `:ref:` for cross-references.
- **Read a how-to that doesn't fit Sphinx** — check `howto/`. These
  are working notes / tips, less polished than the main docs.

## Gotchas

- Two separate Sphinx projects (`docs/` and `jedi-edu/`) — they share
  conventions but build independently. A cross-reference between
  the two requires `intersphinx` or an explicit URL.
- `howto/` content is not rendered into the public site by default.
  Move polished content into `docs/source/...` if you want it
  published.
- The `conf.py` may include a Skylab version-substitution mechanism
  (read `conf.py` before tagging a release; the version may need
  bumping in lockstep with bundle versions).

## Further reading

- Published site: https://jedi-docs.jcsda.org/
- In-repo: `docs/` (read top-level `index.rst`), `jedi-edu/`,
  `howto/`.
- Related briefs: every `jedi-knowledge/<repo>.md` cites jedi-docs
  for canonical user-facing documentation.
