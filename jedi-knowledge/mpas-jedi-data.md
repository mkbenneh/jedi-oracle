# mpas-jedi-data

**Repository:** https://github.com/jcsda-internal/mpas-jedi-data
**Branch tracked by bundle:** develop
**Role in JEDI:** the test-data sidecar for `mpas-jedi` — mesh fixtures and observation files consumed by MPAS-JEDI ctests.

## What it is

A pure data repo (version 3.1.0, "MPAS-JEDI Tier 1 Test Files"). No
source code — the `CMakeLists.txt` is just an
`ecbuild_install_project` declaration. Content lives under
`testinput_tier_1/`.

## How it fits into the bundle

- **Consumed by:** `mpas-jedi` (its `test/CMakeLists.txt` uses this
  repo if found, else a local directory, else a tarball download).
- **Built when:** `BUILD_MPAS=ON` (default ON).
- **Build position:** declared between MPAS-Model and mpas-jedi in
  `jedi-bundle/CMakeLists.txt`.

## Key directories

All under `testinput_tier_1/`:

- `384km/bg/` — 384 km mesh backgrounds: global 2-stream files
  (`mpasout.*.nc`, `x1.4002.invariant.nc`) plus regional CONUS files
  (`conus384km.mpasout.*.nc`, `conus384km.invariant.nc`,
  `conus384km.graph.info`).
- `480km/bg/` — 480 km mesh backgrounds: global 2-stream files
  (`mpasout.*.nc` at three times, `x1.2562.invariant.nc`) plus regional
  CONUS files (`conus480km.*`), and `ensemble/mem01/` … `mem05/` with
  per-member `EnsForCov.*.nc`, `conus480km.EnsForCov.*.nc`, and
  `mpasout.*.nc` files.
- `obs/mpasobsappend1/`, `obs/mpasobsappend2/` — observation files
  (`sondes_obs_2018041500_m.nc4`) appended into the standard test set.

## Key files

- `CMakeLists.txt` — `project( mpas-jedi-data VERSION 3.1.0 ... )` +
  `ecbuild_install_project`.
- The mesh fixtures are the smaller test resolutions (384 km and 480
  km) — convenient for CI but not realistic resolutions.

## Common tasks

- **Skip cloning** — pass `-DBUILD_MPAS=OFF` to the bundle (skips MPAS
  + mpas-jedi + mpas-jedi-data together). Or rely on the tarball
  fallback pinned in `mpas-jedi/CMakeLists.txt` (`3.1.0.jcsda`).
- **Update fixtures** — modify under `testinput_tier_1/`, bump the
  project version, coordinate with mpas-jedi's tarball pin.

## Gotchas

- Directory layout is referenced by literal paths in mpas-jedi YAMLs.
  Renaming silently breaks downstream tests.
- The 2-stream I/O data files (`mpasout.*` / `EnsForCov.*`, replacing
  the old `x1.*.init.*` / `restart.*` convention, repo PR #14) were
  generated with a **single-precision** MPAS build; mpas-jedi `.ref`
  files correspond to this data.
- **Regional CONUS datasets added (repo PR #16):** `conus384km.*` and
  `conus480km.*` files (background, invariant, graph.info, and ensemble
  `conus480km.EnsForCov.*`) support the mpas-jedi regional-mesh ctests.
- **Test dataset configuration corrected (2026-06, repo PR #17):**
  31 NetCDF files across `384km/` and `480km/` were regenerated with a
  corrected configuration, and ensemble members `mem03/`–`mem05/` were
  added. Scripts that reference exact checksums or byte counts for
  these files need regenerating.

## Further reading

- Related briefs: `jedi-knowledge/mpas.md`, `jedi-knowledge/mpas-jedi.md`,
  `jedi-knowledge/jedi-bundle.md`.
