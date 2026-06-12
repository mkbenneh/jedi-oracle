# coupling

**Repository:** https://github.com/jcsda-internal/coupling
**Branch tracked by bundle:** develop
**Role in JEDI:** the atm-ocean coupled DA layer — links `fv3-jedi` and `soca` (and via them, FV3 and MOM6) so JEDI can run coupled atmosphere-ocean assimilation experiments.

## What it is

`coupling` (version 1.2.0, C++ only) combines `fv3-jedi` and `soca`
into coupled OOPS applications using OOPS's `oops/coupled/`
infrastructure: `TraitCoupled<fv3jedi::Traits, soca::Traits>`,
`GeometryCoupled`, `StateCoupled`, `AuxCoupledModel`,
`GetValuesCoupled`, and `BlockDiagonalCovarianceCoupled`.

It is a small repo — there is **no top-level `src/`**; everything
lives under `test_mom6fv3/`. It introduces no new dynamical core: it
composes the two existing model wrappers, and its one library class
(`ModelUFS`) is currently a skeleton (see gotchas).

## How it fits into the bundle

- **Depends on (find_package):** `eckit` (1.24.4), `oops` (1.10.0),
  `jedicmake` (QUIET), `oasim` (QUIET, optional); links `fv3jedi` and
  `soca` (and through them everything those pull: ufo, saber, vader,
  ioda, gsw, fms, MOM6, fv3-jedi-lm, …).
- **Depended on by:** nothing in the bundle; user driver code only.
- **Build position:** last — after fv3-jedi and soca. In the
  jedi-bundle, `BUILD_OASIM` (default OFF) adds the oasim repo, which
  enables the ocean-color HofX test.

## Key directories

- `test_mom6fv3/src/` — `ModelUFS.{cc,h}`, built as library
  `coupled_mom6_fv3` (links eckit, fckit, ioda, oops, fv3jedi, soca).
- `test_mom6fv3/executables/` — the coupled application drivers:
  `TestCoupledGeometry.cc`, `TestCoupledModel.cc`,
  `TestCoupledModelAuxControl.cc`, `forecast_fv3_mom6.cc`,
  `hofx3d_fv3_mom6.cc`, `interp_fv3_mom6.cc`, `var_fv3_mom6.cc`.
- `test_mom6fv3/testinput/` — 12 YAMLs (coupled geometry/model/aux,
  forecasts incl. `forecast_fv3lm_mom6.yaml` and
  `forecast_coupled_fv3_mom6.yaml`, hofx3d variants incl. an
  oasim one and a `dontusemom6` one, interp, var, soca gridgen,
  `fields_metadata_coupled.yml`).
- `test_mom6fv3/testref/` — reference output for the above.
- `bundle/CMakeLists.txt` — coupling's standalone bundle: oasim, oops,
  vader, saber, ioda, gsw, crtm (CRTMv3), ufo, MOM6
  (jcsda-internal fork, `main-ecbuild`), soca, fv3
  (GFDL_atmos_cubed_sphere `release-stable`), femps, fv3-jedi-lm,
  fv3-jedi, fv3-jedi-data — for building only coupling's deps outside
  the larger jedi-bundle.
- `cmake/`, `tools/` — compiler flags and cpplint.

## Key entry points

- `test_mom6fv3/executables/var_fv3_mom6.cc` — coupled variational
  driver: `oops::Variational<oops::TraitCoupled<fv3jedi::Traits,
  soca::Traits>, ufo::ObsTraits>`, with SABER covariance factories
  instantiated per-model and for the coupled trait, plus a
  `BlockDiagonalCovarianceCoupled` maker ("Coupled Block Diagonal").
- `test_mom6fv3/executables/forecast_fv3_mom6.cc` — coupled forecast:
  `oops::Forecast<oops::TraitCoupled<fv3jedi::Traits, soca::Traits>>`.
- `test_mom6fv3/executables/hofx3d_fv3_mom6.cc` — coupled HofX driver.
- `test_mom6fv3/executables/interp_fv3_mom6.cc` — soca-to-fv3 state
  interpolation (uses `GetValuesCoupled`).
- `test_mom6fv3/src/ModelUFS.h` — the placeholder coupled forecast
  model class (`coupled_mom6_fv3::ModelUFS`).

## Common tasks

- **Run the coupled tests** — `ctest -R test_coupled` (or
  `ctest -L coupling`; the directory label is `coupling`). Test names
  are `test_coupled_{geometry,modelaux,model,forecast,
  forecast_coupled,hofx3d,hofx3d_dontusemom6,var}_fv3_mom6`, plus
  `test_soca_gridgen` and `test_interpolate_state_soca_fv3`. They run
  on SOCA's 72x35x25 fixture plus fv3-jedi-data.
- **Add a new coupled application** — drop an executable into
  `test_mom6fv3/executables/`, follow `var_fv3_mom6.cc`
  (instantiate the oops application with
  `oops::TraitCoupled<fv3jedi::Traits, soca::Traits>`), register the
  test in `test_mom6fv3/CMakeLists.txt`.
- **Enable the ocean-color (OASIM) HofX test** — build with the
  bundle option `BUILD_OASIM=ON`; the test
  `test_coupled_hofx3d_oasim_fv3_mom6` is added only when
  `oasim_FOUND`.

## Gotchas

- **`ModelUFS` is a skeleton.** Its `initialize`/`step`/`finalize`
  are TODO stubs — `step` only advances the valid time. Coupled
  forecast tests that actually propagate fields use the per-model
  forecasts via the coupled trait, not ModelUFS. Don't tell users a
  real coupled UFS integration exists here.
- Builds last and inherits every gotcha from fv3-jedi and soca:
  FMS/Atlas/OOPS pinning, MOM6/Icepack submodule recursion, GSW for
  marine recipes, etc.
- The standalone `bundle/CMakeLists.txt` is for building only
  coupling-related deps; in the wider jedi-bundle it's redundant.
  Note it uses the jcsda-internal MOM6 fork and may pin ufo to a
  feature branch — check it before trusting it for a fresh build.
- **FV3 exclude-variables lists updated (2026-05, coupling#76):** the
  test YAMLs in `test_mom6fv3/testinput/` had their excluded FV3
  variable lists updated to follow an fv3-jedi variable-naming change.
  If coupled tests fail on unknown variable names, check that the
  YAMLs are current with develop.

## Further reading

- Related briefs: `jedi-knowledge/fv3-jedi.md`, `jedi-knowledge/soca.md`,
  `jedi-knowledge/oops.md`, `jedi-knowledge/jedi-bundle.md`.
