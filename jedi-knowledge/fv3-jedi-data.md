# fv3-jedi-data

**Repository:** https://github.com/jcsda-internal/fv3-jedi-data
**Branch tracked by bundle:** develop
**Role in JEDI:** the test-data sidecar for `fv3-jedi` — model fixtures, observation files, and configuration data consumed by FV3-side ctests.

## What it is

A data-only ecbuild project (version 1.9.0, ~3.6 GB checked out, files
in git-lfs). It contains no source code — just test fixtures organized
as a tier-1 dataset. The `fv3-jedi` ctest suite consumes it directly
when found via `find_package(fv3-jedi-data QUIET)`; if absent,
`fv3-jedi` falls back to auto-downloading a tarball pinned to tag
1.9.0 from `bin.ssec.wisc.edu`.

The bundle CMakeLists guards inclusion behind
`-DENABLE_FV3_JEDI_DATA=ON` (default ON), so users who don't run tests
can skip cloning it.

## How it fits into the bundle

- **Consumed by:** `fv3-jedi` (its `test/CMakeLists.txt` symlinks
  `testinput_tier_1/inputs` and `testinput_tier_1/obs` into the build
  tree's `Data/` directory).
- **Depends on:** nothing — pure data (CMakeLists is a bare
  `ecbuild_install_project`).
- **Build position:** any time before fv3-jedi tests run.

## Key directories

All content lives under `testinput_tier_1/`:

- `inputs/` — model resolution / experiment fixtures:
  `gfs_c12/`, `gfs_aero_c12/`, `gfs_land_c48/`, `geos_c12/`,
  `geos_4x5/`, `lam_cmaq/`, `lam_rrfs/`, `GEOSChem_c12/`,
  `GEOSChem_c48/`, `femps/`, `fv3files/` (vertical coordinate tables
  `akbk64/72/127/186.nc4`), and `nmcbalance/` (NMC balance regression
  tables — a sibling of `fv3files/`, not inside it).
- `obs/` — ~68 IODA-format observation files (sondes, aircraft,
  satwind, AMSU-A, ATMS, IASI, CrIS, GMI, GNSS-RO, AOD, snow/land,
  SST, …) plus `satbias_*` / aircraft bias-correction files and
  `*_tlapmean.txt` lookup files.

## Key entry points

- `CMakeLists.txt` — single `ecbuild_install_project` declaration
  (project version 1.9.0).
- `testinput_tier_1/inputs/fv3files/akbk*.nc4` — vertical coordinate
  files referenced from many fv3-jedi YAMLs.
- Per-experiment subdirs under `testinput_tier_1/inputs/<flavor>/`
  contain the backgrounds, ensemble members, and surface/restart
  files used by the matching ctests in `fv3-jedi`.

## Common tasks

- **Skip cloning to save space** — pass `-DENABLE_FV3_JEDI_DATA=OFF`
  to the bundle. fv3-jedi's tests will then pull the pinned tarball
  from `bin.ssec.wisc.edu` instead.
- **Update test fixtures** — modify under `testinput_tier_1/`, bump
  the project version in `CMakeLists.txt`, and coordinate with the
  matching `fv3-jedi_data_tag` tarball pin in fv3-jedi's
  `CMakeLists.txt`.

## Gotchas

- Repo is large (~3.6 GB via git-lfs) — pulling the full set takes
  time and disk. Consider the tarball-fallback path if you don't need
  the full tier-1 set locally.
- The directory layout is referenced by literal paths in fv3-jedi
  YAMLs. Renaming directories silently breaks tests downstream.
- Recent data changes worth knowing: infinity values in obs files
  replaced with missing values (#116), CO2 fixtures changed from ppmv
  to mol/mol (#113), project config now generated with ecbuild (#114).

## Further reading

- Related briefs: `jedi-knowledge/fv3-jedi.md`,
  `jedi-knowledge/jedi-bundle.md`, `jedi-knowledge/jedi-model-data.md`.
