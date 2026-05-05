# coupling

**Repository:** https://github.com/jcsda-internal/coupling
**Branch tracked by bundle:** develop
**Role in JEDI:** the atm-ocean coupled DA layer — links `fv3-jedi` and `soca` (and via them, FV3 and MOM6) so JEDI can run coupled atmosphere-ocean assimilation experiments.

## What it is

`coupling` (version 1.2.0) is the JEDI repo that combines `fv3-jedi` and
`soca` into a single coupled OOPS Traits configuration. It uses OOPS's
`coupled/` infrastructure (`GeometryCoupled`, `ModelCoupled`,
`StateCoupled`, `TraitCoupled`) to run a coupled forecast model
(`ModelUFS`) and to drive coupled DA applications.

Think of it as the "fv3 + soca = coupled" repo. It does not introduce a
new dynamical core — it composes the two existing model wrappers.

## How it fits into the bundle

- **Depends on:** `fv3-jedi`, `soca`, `oops`, plus everything those
  pull (ufo, saber, vader, crtm, gsw, fv3-jedi-lm, MOM6, …).
- **Depended on by:** user driver code; not consumed by other bundle
  components.
- **Build position:** after fv3-jedi and soca.

## Key directories

- `src/` — `ModelUFS.{cc,h}` plus coupled-side logic.
- `test/test_mom6fv3/` — integration tests.
  - `test/test_mom6fv3/executables/` —
    `TestCoupledGeometry`, `TestCoupledModel`,
    `TestCoupledModelAuxControl`, `forecast_fv3_mom6`,
    `hofx3d_fv3_mom6`, `interp_fv3_mom6`, `var_fv3_mom6`.
  - Reuses SOCA's 72×35×25 test fixture and fv3-jedi-data tier-1.
- `bundle/CMakeLists.txt` — coupling's own standalone bundle, which
  pulls SOCA + fv3-jedi + GFDL_atmos_cubed_sphere + femps +
  fv3-jedi-linearmodel for builds outside the larger jedi-bundle.

## Key entry points

- `src/ModelUFS.{cc,h}` — the coupled forecast model wrapper.
- `test/test_mom6fv3/executables/var_fv3_mom6.cc` — coupled
  variational driver.
- `test/test_mom6fv3/executables/forecast_fv3_mom6.cc` — coupled
  forecast driver.
- `test/test_mom6fv3/executables/hofx3d_fv3_mom6.cc` — coupled
  HofX driver.
- `bundle/CMakeLists.txt` — the standalone bundle, useful for
  understanding which sub-components coupling needs.

## Common tasks

- **Run the coupled tests** — `ctest -R coupling_` in the build tree.
  These exercise the coupled geometry, model, forecast, HofX, and var
  pipelines on the small SOCA + fv3-jedi-data fixtures.
- **Add a new coupled application** — drop an executable into
  `test/test_mom6fv3/executables/`, follow the pattern of
  `var_fv3_mom6.cc` (instantiate `Variational<TraitCoupled<ATM, OCN>>`).
- **Enable optional ocean-color FO** — coupling has an `oasim` hook
  for OASIM-driven optical observation operators (off by default; see
  the bundle option `BUILD_OASIM`).

## Gotchas

- Builds late and inherits every gotcha from fv3-jedi and soca: FMS /
  Atlas / OOPS pinning, MOM6 submodule recursion, GSW for marine
  recipes, double-precision MPAS irrelevance, etc.
- The standalone `bundle/CMakeLists.txt` is for users who want to
  build *only* coupling-related deps; in the wider jedi-bundle it's
  redundant.
- **FV3 exclude-variables list updated (2026-05, coupling#76):** all
  nine test YAMLs in `test_mom6fv3/testinput/` had their excluded FV3
  variable lists updated to reflect a change in FV3's variable naming.
  If you are running coupled tests and they fail on unknown variable
  names, check that your test YAMLs are current with develop.

## Further reading

- Related briefs: `jedi-knowledge/fv3-jedi.md`, `jedi-knowledge/soca.md`,
  `jedi-knowledge/oops.md`, `jedi-knowledge/jedi-bundle.md`.
