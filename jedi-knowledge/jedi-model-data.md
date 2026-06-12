# jedi-model-data

**Repository:** https://github.com/jcsda-internal/jedi-model-data
**Branch tracked by bundle:** develop
**Role in JEDI:** test-data sidecar for the model-agnostic JEDI components — provides fixtures consumed by `vader` and `saber` tests.

## What it is

A small data-only repo (project version 1.0.0, ~600 MB, NetCDF files
in git-lfs). The `CMakeLists.txt` declares an
`ecbuild_install_project` so downstream CMake projects can locate the
data via `find_package(jedi-model-data 1.0.0 QUIET)`. Content lives
under `testinput_tier_1/`, organized by consumer (`saber/`, `vader/`).

This repo is uncommonly important because vader's and saber's test
suites are *gated* on its presence: when
`find_package(jedi-model-data)` fails, both print "jedi-model-data not
found; skipping ... tests" at configure time and the suites are
silently skipped — builds pass without having actually exercised the
libraries.

## How it fits into the bundle

- **Consumed by:** `vader` (`vader/CMakeLists.txt` ~line 60) and
  `saber` (`saber/CMakeLists.txt` ~line 214) — test data and
  reference inputs for both.
- **Built when:** unconditionally cloned by the bundle (no opt-out
  flag, unlike `fv3-jedi-data`).
- **Build position:** before vader and saber tests.

## Key directories

- `testinput_tier_1/saber/` — saber-side fixtures: covariance/balance
  stats (`CovStats.nc`, `bifourier_*.nc`, `spectralcov.nc`),
  GSI-BEC coefficient files (`gsi-coeffs-*.nc4`) and matching
  `dirac_gsi_*.nml` namelists, `orographic_interp.nc`, `landsea.nc`,
  `gauss_state.nc`, and related stats files.
- `testinput_tier_1/vader/` — vader-side fixtures: input model states
  for variable-change tests (`gauss_state_F12.nc`,
  `aero6_gauss_state_F12.nc`, `rrfs_sd_gauss_state_F12.nc`,
  `dust_gauss_state_F12_lfric2um.nc`,
  `*air_density_levels_minus_one_{A,B}.nc`, `wind_at_10m_A.nc`,
  `wind_at_10m_A_flipped.nc`).

## Key files

- `CMakeLists.txt` — single `ecbuild_install_project` declaration
  (project `jedi-model-data` version 1.0.0).
- `README.md` — points at the consumers (vader and saber).

## Common tasks

- **Update fixtures for a vader or saber test** — modify under the
  appropriate `testinput_tier_1/<consumer>/` subtree. Bump the
  project version if the consumer's `find_package` version
  requirement needs to move with it.
- **Confirm tests are actually running** — at configure time,
  vader/saber print whether `jedi-model-data` was found. Watch for
  "not found; skipping ... tests" messages — they mean tests will be
  silently skipped.

## Gotchas

- Silent test skipping is the main pitfall. If you've made changes to
  vader or saber and "tests still pass," check that
  `jedi-model-data` was found. A missing data repo means the tests
  were never run.
- **Wind product variables added (2026-06, jedi-model-data#14):**
  `testinput_tier_1/vader/wind_at_10m_A.nc` was updated and
  `testinput_tier_1/vader/wind_at_10m_A_flipped.nc` was added to
  support the top-down level ordering tests in vader (see vader#370).
  If your vader test fixtures reference the old `wind_at_10m_A.nc`
  layout, pull this data repo update first.
- **`air_pressure_at_surface` added to `gauss_state_F12.nc` (2026-06,
  jedi-model-data#17):** the vader fixture
  `testinput_tier_1/vader/gauss_state_F12.nc` now carries an
  `air_pressure_at_surface` field, supporting vader recipes that need a
  surface-pressure input. Pull this update before running vader tests
  that reference the new variable.

## Further reading

- Related briefs: `jedi-knowledge/vader.md`, `jedi-knowledge/saber.md`,
  `jedi-knowledge/jedi-bundle.md`.
