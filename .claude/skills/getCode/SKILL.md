---
name: getCode
description: Clone JEDI bundle repositories listed in jedi-bundle/CMakeLists.txt. Asks the user which repos to include and which optional packages to enable, then clones into jedi-bundle/<repo>/.
---

# /getCode

Clone the JEDI bundle sub-repositories into `jedi-bundle/<repo>/`, using
`jedi-bundle/CMakeLists.txt` as the source of truth for repo URLs and
branches.

## Preconditions

- `jedi-bundle/` must already be cloned at the oracle repo root. If not,
  clone it first:
  `git clone https://github.com/jcsda-internal/jedi-bundle.git jedi-bundle`
  (this is normally done by the init flow in CLAUDE.md).

## Flow

1. **Parse `jedi-bundle/CMakeLists.txt`** to enumerate every
   `ecbuild_bundle( PROJECT … GIT … BRANCH/TAG … )` entry. Group them as:
   - **Mandatory** — entries not wrapped in an `option(...)` / `if(...)`
     block (gsw, oops, jedi-model-data, vader, saber, crtm, ioda-data,
     ioda, ufo-data, ufo, fv3-jedi-lm, fv3-jedi-data, fv3-jedi, soca,
     coupling).
   - **Conditional defaults-on** — wrapped in `option(...)` with default
     `ON` (mpas / mpas-jedi / mpas-jedi-data via `BUILD_MPAS`).
   - **Conditional defaults-off** — wrapped in `option(...)` with default
     `OFF` (gsibec, rttov, oasim, ropp-ufo, iodaconv, pyiri-jedi).

2. **Show the user the list** and ask for their selection. Accept any of:
   - "all" — clone everything (mandatory + defaults-on; ask separately
     about defaults-off optionals)
   - a whitelist — "just oops, ufo, ioda"
   - a blacklist — "everything except mpas-jedi-data"

3. **Ask about optional packages** that default to OFF:
   *"Want to enable any optional packages? RTTOV, OASIM, ROPP, GSIBEC,
   IODA-converters, PyIRI."* For each one the user enables, edit
   `jedi-bundle/CMakeLists.txt` to flip the corresponding
   `option(BUILD_<NAME> "..." OFF)` to `ON`. Tell the user you've modified
   their CMakeLists so they can review.

4. **Clone the selected repos** into `jedi-bundle/<repo>/`. For each:
   - Skip with a note if the directory already exists and is a git repo.
   - Use the URL from CMakeLists.
   - Use the configured `BRANCH` or `TAG`. If `RECURSIVE` is set in the
     CMakeLists entry, pass `--recursive` to `git clone`.
   - Run `git lfs version` first if cloning anything that uses LFS
     (currently none of the bundle repos require it, but jedi-tools does
     — that's handled in the init flow, not here).

5. **Report** a per-repo result: cloned / skipped (already present) /
   failed (with error). End with a one-line summary.

## Notes

- This skill modifies `jedi-bundle/CMakeLists.txt` if the user enables
  optional packages. Those edits are inside the gitignored `jedi-bundle/`
  clone, so they don't affect the oracle repo. Mention this so the user
  knows where the change is.
- This skill does **not** clone `jedi-docs/`, `jedi-tools/`, or
  `jedi-workflow/` — those are handled by the init flow in CLAUDE.md.
- After cloning, suggest the user run a build (or point them at
  `jedi-tips.md` and `jedi-docs/` for build instructions) — but don't
  build for them unless asked.
