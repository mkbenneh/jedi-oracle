# fv3-jedi-lm

**Repository:** https://github.com/jcsda-internal/fv3-jedi-linearmodel
**Branch tracked by bundle:** develop
**Role in JEDI:** the FV3 linear model — tangent-linear and adjoint of the FV3 dycore + linearized physics, used by `fv3-jedi`'s `Tlm/` to enable 4DVar.

## What it is

`fv3-jedi-lm` is the Fortran-only library that provides the tangent
linear and adjoint of the FV3 dynamical core plus a set of linearized
physics packages. It's a long-lived companion to the operational FV3
nonlinear code, kept current with TLM/AD pairs so JEDI's 4DVar (in
`oops`) can run with FV3 as the model.

`fv3-jedi`'s `src/fv3jedi/Tlm/` consumes this library to satisfy OOPS's
`LinearModelBase` interface.

## How it fits into the bundle

- **Depends on:** Fortran compiler, MPI, NetCDF, fckit. No other JEDI
  repos.
- **Depended on by:** `fv3-jedi` (`find_package(fv3jedilm >= 1.5)`).
- **Build position:** before fv3-jedi.

## Key directories

- `src/` — top-level Fortran sources.
  - `fv3jedi_lm_mod.F90` — top-level type combining
    `fv3jedi_lm_dynamics_type` and `fv3jedi_lm_physics_type` with the
    `init_nl/tl/ad`, `step_nl/tl/ad`, `final_nl/tl/ad` cycle.
- `src/dynamics/atmos_cubed_sphere/` — embedded copy of the FV3 dycore
  split into:
  - `model/` — nonlinear sources (`*_nlm.F90`).
  - `model_tlmadm/` — TLM/AD pairs (`*_tlm.F90` / `*_adm.F90`).
  - `tools/` — supporting utilities.
- `src/physics/` — GEOS-derived linearized physics:
  - `moist/` (cloud, convection)
  - `turbulence/` (`bldriver`, `blsimp`)
  - `gwd/` (gravity-wave drag)
  - `radiation/`
- `src/utils/` — `fv3jedi_lm_utils_mod.F90`,
  `fv3jedi_lm_const_mod.F90`, `MAPL_Constants.F90`, `tapenade/` (some
  derivatives are generated with Tapenade).

## Key entry points

- `src/fv3jedi_lm_mod.F90` — the top-level type and lifecycle methods.
- `src/dynamics/atmos_cubed_sphere/model_tlmadm/` — the TLM/AD dycore
  routines; pair `*_tlm.F90` files with their `*_adm.F90` partners.
- `src/physics/turbulence/blsimp_*.F90`,
  `src/physics/moist/cloud_*.F90` — examples of linearized physics
  modules.

## Common tasks

- **Build standalone** — `mkdir build && cd build && ecbuild ../ &&
  make -j` (no JEDI prerequisites).
- **Verify TL/AD consistency** — fv3-jedi's `Tlm/` ctest suite
  exercises the linearization at the JEDI level. fv3-jedi-lm itself
  has Fortran-level checks for some routines.
- **Add a new linearized physics module** — add `*_tlm.F90` /
  `*_adm.F90` under `src/physics/<group>/`, update the matching
  CMakeLists, expose via `fv3jedi_lm_physics_type` if it should be
  toggled at runtime.

## Gotchas

- TLM/AD pairs must remain numerically consistent with the nonlinear
  code. When updating either side, run the JEDI-level TL test in
  fv3-jedi to confirm the linearization still holds at the expected
  precision.
- Some derivatives are generated with Tapenade (see `src/utils/tapenade/`)
  — regenerating requires an external Tapenade install.
- Embedded FV3 dycore copy can drift from upstream FV3-GFS — check
  before bumping FMS / cubed-sphere upstream.
- **FMS 2025.2 compatibility (2026-06, fv3-jedi-lm#39):** Features not
  present in FMS 2025.2 were removed across `src/dynamics/fv3jedi_lm_dynamics_mod.F90`
  and nine `atmos_cubed_sphere/tools/*.F90` files (fv_mp_nlm_mod, fv_restart_nlm,
  fv_treat_da_inc_nlm, and others). If you build against an older FMS, this
  may require reverting or adapting those removals.

## Further reading

- Related briefs: `jedi-knowledge/fv3-jedi.md`,
  `jedi-knowledge/oops.md`, `jedi-knowledge/jedi-bundle.md`.
