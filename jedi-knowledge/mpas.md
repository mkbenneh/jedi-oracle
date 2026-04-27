# mpas (MPAS-Model)

**Repository:** https://github.com/MPAS-Dev/MPAS-Model
**Pin tracked by bundle:** commit `0e5a47a0e1bcccd6e3d99909b76e740a643c4db6` on the `release-v8.4.0` branch
**Role in JEDI:** the MPAS atmospheric forecast model itself — the upstream NCAR/LANL MPAS-Atmosphere code that `mpas-jedi` wraps for JEDI integration.

## What it is

MPAS (Model for Prediction Across Scales) is the unstructured-mesh
weather and climate model maintained by NCAR and LANL. The bundle pulls
its `Atmosphere` core (and optionally `init_atmosphere` for
preprocessing) so that `mpas-jedi` has a real forecast model to wrap.
Other MPAS cores (`core_ocean`, `core_seaice`, `core_landice`, `core_sw`,
`core_test`) ship in the upstream repo but the bundle does not build
them — see the `MPAS_CORES` setting in jedi-bundle's CMakeLists
(`init_atmosphere atmosphere`).

## How it fits into the bundle

- **Built when:** `BUILD_MPAS=ON` (default ON).
- **Built with:** `MPAS_DOUBLE_PRECISION=ON`, `MPAS_OPENMP=ON`,
  `MPAS_CORES="init_atmosphere atmosphere"` (defaults set in
  `jedi-bundle/CMakeLists.txt`).
- **Depended on by:** `mpas-jedi`. The bundle builds MPAS first, then
  `mpas-jedi`, then `mpas-jedi-data`.

## Key files

- `CMakeLists.txt` — entry point for the cmake build path the bundle
  uses.
- `src/core_atmosphere/` — atmospheric solver (the bundled core).
- `src/core_init_atmosphere/` — initial-condition / preprocessing
  core (also built when bundle sets `MPAS_CORES`).
- `src/core_ocean/`, `src/core_seaice/`, `src/core_landice/`,
  `src/core_sw/`, `src/core_test/` — other cores; **not** built by
  the bundle.
- `Registry.xml` (per core) — declarative variable / time-stepping
  config that drives MPAS code generation.

## Common tasks

- **Build via the bundle** — automatic when `BUILD_MPAS=ON`; see the
  flags listed above.
- **Add another MPAS core to the bundle build** — set
  `MPAS_CORES="init_atmosphere atmosphere ocean"` in
  `jedi-bundle/CMakeLists.txt` (and ensure required deps are
  available).

## Gotchas

- **Pinned to a commit, not a tag.** The bundle CMakeLists comment
  explains why: *"Using a commit from MPAS-model release-v8.4.0
  branch which contains the fix for MPAS-model#1420. Once an official
  tag with this fix exists we should use that instead."* Implications:
  - `/updateRepos` skips this repo (it would `git pull` on the
    pinned commit, not move forward).
  - To pull updates from upstream, edit the `TAG` field in
    `jedi-bundle/CMakeLists.txt` to a newer commit or tag.
  - Don't `git checkout develop` inside the cloned `mpas/` and expect
    the bundle to still work — the fix for #1420 lives on the
    release branch.
- The MPAS build requires a working PIO/PNETCDF or NetCDF4 stack with
  parallel I/O. Module setup at HPC sites usually handles this; on
  local machines it's a frequent build blocker.

## Further reading

- https://mpas-dev.github.io/ — upstream docs.
- Related briefs: `jedi-knowledge/mpas-jedi.md`,
  `jedi-knowledge/mpas-jedi-data.md`, `jedi-knowledge/jedi-bundle.md`.
