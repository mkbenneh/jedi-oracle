# ioda-data

**Repository:** https://github.com/jcsda-internal/ioda-data
**Branch tracked by bundle:** develop
**Role in JEDI:** the test-data sidecar for `ioda` — observation files (NetCDF4/HDF5, ODB, BUFR) used by IODA's ctests.

## What it is

A pure data repo. The single `CMakeLists.txt` declares project
`ioda-data` (version 2.9.0, matching ioda's version) and just does
`ecbuild_install_project`; all content lives under `testinput_tier_1/`
(~145 files plus a few subdirectories such as `test_reference/` and
`test_append*/`). The bundle gates it on `ENABLE_IODA_DATA=ON`
(default ON).

If ioda-data is not found at configure time, ioda's `test/CMakeLists.txt`
falls back to (in order): a local directory pointed to by the
`IODA_TESTFILES` environment variable, then a versioned tarball download
(`ioda_testinput_tier_1_<tag>.tar.gz`, tag/URL/hash set in ioda's
top-level `CMakeLists.txt`).

## How it fits into the bundle

- **Consumed by:** `ioda` only — its `test/` suite symlinks
  `testinput_tier_1/` into the build tree as `Data/testinput_tier_1`.
- **Build position:** listed before `ioda` in
  `jedi-bundle/CMakeLists.txt` so `find_package(ioda-data)` succeeds.

## Key directories

- `testinput_tier_1/` — flat collection of test files:
  - `*.nc4` / `*.nc` — IODA-format obs files by instrument/type prefix
    (`amsua_*`, `sondes_*`, `gnssro_*`, `aod_*`, `smap_*`, …),
    including multi-part variants (`*_p1..p3`) for multi-file-read
    tests, `obsspace_*` fixtures (grouping, reduce, osdf), `upgrader_*`
    inputs for the v2→v3 upgrader tests, and edge cases
    (`empty_obs_file.nc4`, `zero_obs_v2.nc4`, `invalid_numeric_test.nc4`).
  - `*.odb` — ODB files per obs type (`sonde.odb`, `atms.odb`,
    `satwind.odb`, …) for the ODC engine tests driven by
    `ioda/share/ioda/yaml/iodatest_odb_*.yaml`.
  - `*.bufr_d` — BUFR engine test input.
  - `test_reference/` — reference outputs for comparison tests;
    `test_append1..3/` — append-test fixtures.

Note: geovals/obsdiag files live in `ufo`'s test data, not here —
ioda-data holds only obs-side files.

## Common tasks

- **Skip cloning to save space** — pass `-DENABLE_IODA_DATA=OFF` to
  the bundle; ioda tests then use the tarball (or `IODA_TESTFILES`).
- **Update fixtures** — add/modify files under `testinput_tier_1/`;
  reference-file updates usually pair with an ioda PR (e.g. the
  `dimList`, datetime-units, and ODB-Null changes each landed matching
  ioda-data PRs).

## Gotchas

- File names are referenced by literal strings in ioda's test YAMLs
  (`ioda/test/testinput/` and `ioda/share/ioda/yaml/`); renaming
  silently breaks tests.
- The tarball fallback is pinned to a release tag, so a develop-branch
  ioda paired with the tarball (instead of the repo) can fail tests
  whose reference files changed since that tag. When ioda tests fail
  on reference comparisons, check that ioda and ioda-data are in sync.
- Recent reference updates (2026): GMI-GPM references for the new ioda
  OSDF `time_io` tests, `gmi_gpm_obs_*_time_osdf_mpi{1,2,4}*.nc4` under
  `test_reference/` (#245, pairs with ioda#1773); ODB writes Null
  instead of missing (#243); `put_db` dimList changes (#242); datetime
  units now epoch-style `seconds since 1970-01-01T00:00:00Z` (#241).

## Further reading

- Related briefs: `jedi-knowledge/ioda.md`, `jedi-knowledge/ufo.md`,
  `jedi-knowledge/jedi-bundle.md`.
