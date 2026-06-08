# fv3-jedi

**Repository:** https://github.com/jcsda-internal/fv3-jedi
**Branch tracked by bundle:** develop
**Role in JEDI:** the FV3-side OOPS Traits implementation — adapts FV3-based atmospheric models (FV3-GFS, GEOS, UFS) to JEDI so they can be driven by oops applications (Variational, HofX, Forecast, EnKF, …).

## What it is

`fv3-jedi` is the model interface that wraps the FV3 family of dynamical
cores (NOAA FV3-GFS, NASA GEOS, NOAA UFS) so JEDI's generic `oops::`
algorithms can drive them. It implements every Traits component an OOPS
application requires: `Geometry`, `State`, `Increment`, `Fields`,
`Model`, `LinearModel` (via fv3-jedi-lm), `VariableChange`,
`LinearVariableChange`, `ErrorCovariance`, `ModelBias`,
`GeometryIterator`, `IO` backends.

CMake options choose which FV3 forecast model to build the interface
for: `FV3_FORECAST_MODEL` (FV3CORE / GEOS / UFS), and
`UFS_APP` (ATM / ATMAERO / S2S) when UFS is selected.

## How it fits into the bundle

- **Depends on:** `oops >= 1.10`, `ufo >= 1.10`, `saber >= 1.10`,
  `vader >= 1.7`, `crtm >= 2.2.3`, `atlas >= 0.35`, `fms 2023.04`,
  `fv3jedilm >= 1.5` (the linear model from `fv3-jedi-lm`).
- **Depended on by:** `coupling` (combines fv3-jedi + soca for
  atm-ocean DA) and any user driver code that links FV3 with JEDI.
- **Build position:** late in the bundle (after oops, ufo, saber,
  vader, crtm, fv3-jedi-lm, and the FMS chosen by the bundle).

## Key directories

- `src/fv3jedi/` — one subdirectory per OOPS Traits component:
  `Geometry/`, `State/`, `Increment/`, `Fields/`, `FieldMetadata/`,
  `Model/{fv3lm,geos,ufs,pseudo}/`, `Tlm/`, `IO/{FV3Restart,
  CubeSphereHistory,StructuredGrid}/`, `VariableChange/`,
  `LinearVariableChange/`, `ErrorCovariance/`, `ModelBias/`,
  `NormGradient/`, `ObsLocalization/`, `GeometryIterator/`,
  `ModelData/`, `Utilities/`.
- `src/mains/` — ~27 application drivers that instantiate `oops::`
  applications with `FV3JEDITraits`. These produce the model
  executables (e.g. `fv3jedi_var.x`, `fv3jedi_hofx.x`,
  `fv3jedi_forecast.x`, `fv3jedi_eda.x`, `fv3jedi_letkf.x`,
  `fv3jedi_convertstate.x`, `fv3jedi_genensptb.x`, …).
- `test/` — ~199 test YAMLs and ~146 reference outputs covering 3DVar,
  4DVar, EnVar, HofX, EnKF/LETKF, ConvertState, IO round-trips, and
  variable-change checks. Test data is auto-fetched from a tarball on
  `bin.ssec.wisc.edu` (pinned to v1.9.0) when `fv3-jedi-data` is not
  found.
- `docs/` — design notes and tutorials.

## Key entry points

- `src/fv3jedi/Traits.h` — the `FV3JEDITraits` struct that ties all the
  FV3 component implementations together for OOPS templates.
- `src/mains/fv3jediVar.cc`, `fv3jediHofX4D.cc`, `fv3jediForecast.cc`,
  `fv3jediEDA.cc`, `fv3jediLETKF.cc` — canonical executable templates.
- `src/fv3jedi/Geometry/Geometry.{h,cc}` — start here to see how an
  FV3 cubed-sphere grid is wrapped for JEDI/Atlas.
- `src/fv3jedi/State/State.{h,cc}`, `Increment/Increment.{h,cc}`,
  `Fields/Fields.{h,cc}` — the core state representation.
- `src/fv3jedi/Model/fv3lm/`, `Model/geos/`, `Model/ufs/` — per-flavor
  forecast model wrappers selected by `FV3_FORECAST_MODEL`.
- `src/fv3jedi/IO/FV3Restart/`, `CubeSphereHistory/`,
  `StructuredGrid/` — the three I/O backends.
- `src/fv3jedi/FieldMetadata/` — variable metadata (long names,
  units, vertical levels) consulted by VariableChange and IO.

## Common tasks

- **Run the FV3-GFS reference tests** — configure with
  `-DFV3_FORECAST_MODEL=FV3CORE`, then `ctest -R fv3jedi_`.
- **Switch to GEOS** — `-DFV3_FORECAST_MODEL=GEOS` (requires GEOS
  dependencies in the env).
- **Switch to UFS** — `-DFV3_FORECAST_MODEL=UFS -DUFS_APP=ATM` (or
  `ATMAERO` / `S2S`).
- **Add a new variable change** — drop a new class under
  `src/fv3jedi/VariableChange/`, register in the factory, add a test
  YAML in `test/testinput/`.
- **Run a 3DVar experiment** — typically driven by the workflow stack
  via skylab + ewok; the executable is `fv3jedi_var.x` invoked with a
  YAML.

## Gotchas

- The build is sensitive to FMS / Atlas / OOPS versions — bumping any
  of these often requires fv3-jedi changes too (the `>=` pins are
  conservative, not aspirational).
- Three different forecast models are gated behind one CMake option:
  builds for one are not interchangeable with another's executables.
  Choose `FV3_FORECAST_MODEL` deliberately.
- The auto-tarball fallback for test data is convenient but pinned to
  a specific version (v1.9.0). Bumping fv3-jedi means also bumping the
  tarball URL in `CMakeLists.txt` (or providing a local
  `fv3-jedi-data` checkout).
- 4DVar requires the linear model — `fv3-jedi-lm` must be built and
  found via `find_package(fv3jedilm)`.
- **CRTM is now optional (2026-06, fv3-jedi#1506):** `CMakeLists.txt`
  gates CRTM on an optional find, so fv3-jedi builds succeed without CRTM
  present. If your configuration relied on CRTM being unconditionally
  linked, add `find_package(crtm)` explicitly or pass `-DWITH_CRTM=ON`.
- **FMS2-only IO (2026-06, fv3-jedi#1509):** remaining non-FMS2 I/O code
  was removed from `src/fv3jedi/IO/FV3Restart/fv3jedi_io_fms2_mod.f90` and
  the main drivers. Builds now require FMS ≥ 2025.2.
- **Level ordering (2026-06, fv3-jedi#1510):** a "levels are top down?"
  fix in `src/fv3jedi/ModelData/ModelData.cc/.h` affects convertstate
  reference outputs (`test/testoutput/convertstate_*.ref` all updated).
  If your branch has local ref-output edits, expect merge conflicts there.

## Further reading

- jedi-docs.jcsda.org → JEDI Components → fv3-jedi.
- Related briefs: `jedi-knowledge/oops.md`, `jedi-knowledge/ufo.md`,
  `jedi-knowledge/saber.md`, `jedi-knowledge/vader.md`,
  `jedi-knowledge/crtm.md`, `jedi-knowledge/fv3-jedi-data.md`,
  `jedi-knowledge/fv3-jedi-lm.md`, `jedi-knowledge/coupling.md`,
  `jedi-knowledge/jedi-bundle.md`.
