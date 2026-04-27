# ufo-data

**Repository:** https://github.com/jcsda-internal/ufo-data
**Branch tracked by bundle:** develop
**Role in JEDI:** the test-data sidecar for `ufo` — IODA-format observations and matching geoVaLs / obs-diagnostics used by UFO's instrument and unit tests.

## What it is

A pure data repo. The CMakeLists declares an `ecbuild_install_project`;
content lives under `testinput_tier_1/`. The bundle gates this on
`ENABLE_UFO_DATA=ON` (default ON).

This is the largest of the data sidecars — UFO's test matrix covers many
instruments and operator variants, so the fixture set is
correspondingly large (on the order of ~647 files per scan results).

## How it fits into the bundle

- **Consumed by:** `ufo` (its `test/` suite reads files under
  `testinput_tier_1/`).
- **Build position:** before `ufo` tests run.

## Key directories

- `testinput_tier_1/` — split by content category and instrument
  prefix:
  - `*_obs_*` — observation values.
  - `*_geoval_*` — geovals (model values interpolated to obs
    locations).
  - `*_obsdiag_*` — observation diagnostics.
  - bias-correction (`satbias_*`, `acftbias_*`) and covariance files.
  - Per-instrument grouping (sondes, aircraft, AMSU-A/B, ATMS, IASI,
    CrIS, AIRS, ABI, AHI, GMI, AMSR2, SSMIS, MHS, GNSS-RO, AOD, SMAP,
    SST, scatwind, etc.).

## Common tasks

- **Skip cloning to save space** — pass `-DENABLE_UFO_DATA=OFF` to
  the bundle. UFO tests will then use the tarball fallback path.
- **Update fixtures** — modify under `testinput_tier_1/`, bump the
  project version in `CMakeLists.txt`.

## Gotchas

- This is a large repo — full clone takes time and disk. The
  `ENABLE_UFO_DATA=OFF` path is genuinely useful when you don't need
  to run UFO tests.
- File naming conventions are referenced by literal strings in UFO's
  test YAMLs; renaming silently breaks tests.

## Further reading

- Related briefs: `jedi-knowledge/ufo.md`, `jedi-knowledge/ioda-data.md`,
  `jedi-knowledge/jedi-bundle.md`.
