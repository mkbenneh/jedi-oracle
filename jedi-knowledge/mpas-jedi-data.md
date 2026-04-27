# mpas-jedi-data

**Repository:** https://github.com/jcsda-internal/mpas-jedi-data
**Branch tracked by bundle:** develop
**Role in JEDI:** the test-data sidecar for `mpas-jedi` — mesh fixtures and observation files consumed by MPAS-JEDI ctests.

## What it is

A pure data repo (version 3.1.0). No source code — the `CMakeLists.txt`
is just an `ecbuild_install_project` declaration. Content lives under
`testinput_tier_1/`.

## How it fits into the bundle

- **Consumed by:** `mpas-jedi` (its `test/` suite uses paths under
  `testinput_tier_1/`).
- **Built when:** `BUILD_MPAS=ON` (default ON).
- **Build position:** between MPAS-Model and mpas-jedi.

## Key directories

All under `testinput_tier_1/`:

- `384km/init/` — 384 km global mesh initial conditions.
- `480km/bg/` — 480 km global mesh backgrounds.
- `480km_2stream/` — 480 km mesh with 2-stream radiation fixtures.
- `obs/mpasobsappend1/`, `obs/mpasobsappend2/` — observation files
  appended into the standard test set.

## Key files

- `CMakeLists.txt` — `ecbuild_install_project(... VERSION 3.1.0)`.
- The mesh fixtures are the smaller test resolutions (384 km and 480
  km globally) — convenient for CI but not realistic resolutions.

## Common tasks

- **Skip cloning** — pass `-DBUILD_MPAS=OFF` to the bundle (skips MPAS
  + mpas-jedi + mpas-jedi-data together). Or use the tarball fallback
  pinned in `mpas-jedi/CMakeLists.txt` (`3.1.0.jcsda`) by leaving
  this repo unfetched.
- **Update fixtures** — modify under `testinput_tier_1/`, bump the
  project version, coordinate with mpas-jedi's tarball pin.

## Gotchas

- Directory layout is referenced by literal paths in mpas-jedi YAMLs.
  Renaming silently breaks downstream tests.

## Further reading

- Related briefs: `jedi-knowledge/mpas.md`, `jedi-knowledge/mpas-jedi.md`,
  `jedi-knowledge/jedi-bundle.md`.
