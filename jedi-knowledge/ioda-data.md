# ioda-data

**Repository:** https://github.com/jcsda-internal/ioda-data
**Branch tracked by bundle:** develop
**Role in JEDI:** the test-data sidecar for `ioda` — IODA-format observation files used by IODA's ctests.

## What it is

A pure data repo. Single CMakeLists declares an `ecbuild_install_project`;
content lives entirely under `testinput_tier_1/`. The bundle gates this
on `ENABLE_IODA_DATA=ON` (default ON) — when off, IODA tests can fall
back to a tarball-based fixture.

## How it fits into the bundle

- **Consumed by:** `ioda` (its `test/` suite reads files under
  `testinput_tier_1/`).
- **Build position:** before `ioda` tests run.

## Key directories

- `testinput_tier_1/` — the curated set of IODA-format test files.
  Files are split by content category:
  - `*_obs_*` — observation values.
  - `*_geoval_*` — geovals (model values interpolated to obs
    locations).
  - `*_obsdiag_*` — observation diagnostics.
  - bias-correction and covariance files.
  - Files are grouped by instrument / type prefix.

Total count is on the order of ~145 files (per scan results).

## Common tasks

- **Skip cloning to save space** — pass `-DENABLE_IODA_DATA=OFF` to
  the bundle. IODA tests will then use the alternate tarball
  source.
- **Update fixtures** — modify under `testinput_tier_1/`, bump the
  project version in `CMakeLists.txt`.

## Gotchas

- File naming conventions are referenced by literal strings in IODA's
  test YAMLs; renaming silently breaks tests.

## Further reading

- Related briefs: `jedi-knowledge/ioda.md`, `jedi-knowledge/ufo.md`,
  `jedi-knowledge/jedi-bundle.md`.
