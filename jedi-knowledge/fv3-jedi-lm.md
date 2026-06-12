# fv3-jedi-lm

**Repository:** https://github.com/jcsda-internal/fv3-jedi-linearmodel
**Branch tracked by bundle:** develop (bundle project name `fv3-jedi-lm`)
**Role in JEDI:** the FV3 linear model — tangent-linear and adjoint of the FV3 dycore + linearized physics, used by `fv3-jedi`'s `Tlm/` to enable 4DVar.

## What it is

`fv3-jedi-lm` (CMake project `fv3jedilm`, version 1.5.0) is the
Fortran-only library that provides the tangent linear and adjoint of
the FV3 dynamical core plus a set of GEOS-derived linearized physics
packages. It's a long-lived companion to the operational FV3 nonlinear
code, kept current with TLM/AD pairs so JEDI's 4DVar (in `oops`) can
run with FV3 as the model.

`fv3-jedi`'s `src/fv3jedi/Tlm/` consumes this library to satisfy OOPS's
`LinearModel` trait.

## How it fits into the bundle

- **Depends on:** Fortran compiler, MPI, NetCDF (Fortran), LAPACK
  (MKL preferred, `-DENABLE_MKL=Off` to force plain LAPACK), OpenMP,
  `FMS >= 2023.04` (R4+R8). No other JEDI repos (jedicmake is found
  QUIET for the NetCDF find module). A GEOS build
  (`FV3_FORECAST_MODEL MATCHES GEOS`) instead pulls MAPL/GEOSgcm from
  `FV3_FORECAST_MODEL_ROOT`.
- **Depended on by:** `fv3-jedi` (`find_package(fv3jedilm 1.5.0
  REQUIRED)`).
- **Build position:** before fv3-jedi.

## Key directories

- `src/` — top-level Fortran sources; no `test/` directory exists in
  this repo (verification happens at the fv3-jedi level).
  - `fv3jedi_lm_mod.F90` — top-level `fv3jedi_lm_type` holding `conf`,
    `pert`, `traj`, a `fv3jedi_lm_dynamics_type`, and a
    `fv3jedi_lm_physics_type`, with the `init_nl/tl/ad`,
    `step_nl/tl/ad`, `final_nl/tl/ad` lifecycle.
- `src/dynamics/` — `fv3jedi_lm_dynamics_mod.F90`,
  `fv3jedi_lm_dynutils_mod.f90`, and `atmos_cubed_sphere/`, an
  embedded copy of the FV3 dycore split into:
  - `model/` — nonlinear sources (`*_nlm.F90`).
  - `model_tlmadm/` — TLM/AD pairs (`*_tlm.F90` / `*_adm.F90`, e.g.
    `dyn_core_tlm.F90` / `dyn_core_adm.F90`, `fv_dynamics_*`,
    `fv_mapz_*`), plus shared `*_tlmadm.F90` (fv_arrays, fv_control).
  - `tools/` — supporting utilities (`fv_restart_nlm.F90`,
    `fv_treat_da_inc_nlm.F90`, `fv_io_nlm.F90`, …).
- `src/physics/` — `fv3jedi_lm_physics_mod.F90` plus GEOS-derived
  linearized physics:
  - `moist/` — `cloud.F90`/`cloud_tl.F90`/`cloud_ad.F90`,
    `convection.F90`/`_tl`/`_ad`, `qsat_util.F90`,
    `fv3jedi_lm_moist_mod.F90`.
  - `turbulence/` — `bldriver.F90`, `blsimp.F90`,
    `fv3jedi_lm_turbulence_mod.F90`.
  - `gwd/` — `gw_drag_init.F90`, `gw_drag_d.F90` / `gw_drag_b.F90`
    (Tapenade forward/backward naming).
  - `radiation/` — `irrad`/`sorad` with `_tl`/`_ad` pairs and
    constants modules.
- `src/utils/` — `fv3jedi_lm_utils_mod.F90`,
  `fv3jedi_lm_const_mod.F90`, `fv3jedi_lm_kinds_mod.F90`,
  `MAPL_Constants.F90`, and `tapenade/` (`adStack.c`, `adBuffer.f`,
  `tapenade_iter.F90` — runtime support for Tapenade-generated
  derivatives).

## Key entry points

- `src/fv3jedi_lm_mod.F90` — the top-level type and lifecycle methods.
- `src/dynamics/atmos_cubed_sphere/model_tlmadm/` — the TLM/AD dycore
  routines; pair `*_tlm.F90` files with their `*_adm.F90` partners.
- `src/physics/fv3jedi_lm_physics_mod.F90` — where physics packages
  are toggled and sequenced.
- `src/physics/moist/cloud_tl.F90` / `cloud_ad.F90` — example of a
  linearized physics TL/AD pair.

## Common tasks

- **Build standalone** — `mkdir build && cd build && ecbuild ../ &&
  make -j` (needs FMS, NetCDF, LAPACK/MKL, MPI; no JEDI repos).
- **Verify TL/AD consistency** — fv3-jedi's `Tlm/` ctest suite
  exercises the linearization at the JEDI level; this repo has no
  ctests of its own.
- **Add a new linearized physics module** — add the TL/AD sources
  under `src/physics/<group>/` (follow the local naming: `_tl`/`_ad`
  in moist/radiation, `_d`/`_b` for Tapenade-generated gwd), update
  the matching CMakeLists, expose via `fv3jedi_lm_physics_type` if it
  should be toggled at runtime.

## Gotchas

- TLM/AD pairs must remain numerically consistent with the nonlinear
  code. When updating either side, run the JEDI-level TLM tests in
  fv3-jedi to confirm the linearization still holds at the expected
  precision.
- Some derivatives are generated with Tapenade (runtime support in
  `src/utils/tapenade/`) — regenerating requires an external Tapenade
  install.
- Embedded FV3 dycore copy can drift from upstream FV3-GFS — check
  before bumping FMS / cubed-sphere upstream.
- **FMS 2025.2 alignment (2026-06, fv3-jedi-lm#39):** features not
  present in FMS 2025.2 were removed across
  `src/dynamics/fv3jedi_lm_dynamics_mod.F90` and several
  `atmos_cubed_sphere/tools/*.F90` files (fv_mp_nlm, fv_restart_nlm,
  fv_treat_da_inc_nlm, and others). The CMake minimum is still FMS
  2023.04, but building against an older FMS may require reverting or
  adapting those removals. The preceding PR (#37) moved remaining IO
  to FMS2-IO subroutines.
- fv3-jedi no longer carries staggered (D-grid) variables (#33) — the
  LM handles D-grid winds internally and can be initialized from
  A-grid winds via `initialize_dwinds_from_awinds` in
  `src/fv3jedi_lm_mod.F90`.

## Further reading

- Related briefs: `jedi-knowledge/fv3-jedi.md`,
  `jedi-knowledge/oops.md`, `jedi-knowledge/jedi-bundle.md`.
