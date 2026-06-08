# mpas-jedi

**Repository:** https://github.com/jcsda-internal/mpas-jedi
**Branch tracked by bundle:** develop
**Role in JEDI:** the MPAS-side OOPS Traits implementation — adapts the MPAS-Atmosphere model to JEDI so it can be driven by oops applications (Variational, HofX, Forecast, EnKF, …).

## What it is

`mpas-jedi` (project name `mpasjedi` in CMake) is the model interface
that wraps NCAR/LANL's MPAS-Atmosphere model for JEDI. Like `fv3-jedi`
and `soca`, it implements the OOPS Traits surface so generic
`oops::` algorithms run on MPAS configurations. Version 3.1.0.

CMake requires `MPAS 8.0` with the `core_atmosphere` core. By default
double precision is enabled (`DOUBLE_PRECISION=ON`).

## How it fits into the bundle

- **Depends on:** the cloned `mpas` repo (MPAS-Model with at least
  `core_atmosphere`), `oops`, `ufo`, `saber`, `vader`, `crtm`.
- **Depended on by:** user driver code; not consumed by other bundle
  components.
- **Build position:** after MPAS-Model and the JEDI framework.

## Key directories

- `src/mpasjedi/` — the MPAS-JEDI library, organized into one subdir
  per Traits component:
  `Geometry/`, `State/`, `Increment/`, `Model/`, `Tlm/`, `Covariance/`,
  `IO/`, `VariableChange/`, `LinearVariableChange/`, `ObsLocalization/`,
  `GeometryIterator/`, plus internals.
- `src/mains/` — application drivers: `mpasVariational.cc`,
  `mpasHofX.cc`, `mpasEDA.cc`, `mpasEnKF.cc`, `mpasSACA.cc`, … Producing
  executables consumed by skylab/ewok experiments.
- `test/` — 70+ test YAMLs and reference outputs.
- `workflow/` — csh scripts for running experiments outside of EWOK
  (legacy / standalone).

## Key entry points

- `src/mpasjedi/Traits.h` — the `MPASJEDITraits` struct.
- `src/mains/mpasVariational.cc` — canonical 3DVar / EnVar / 4DVar
  driver for MPAS.
- `src/mains/mpasHofX.cc` — HofX (forward operator) driver.
- `src/mpasjedi/Geometry/Geometry.{h,cc}` — wraps the MPAS
  unstructured mesh for Atlas / OOPS.
- `src/mpasjedi/Model/Model.{h,cc}` — MPAS forecast model wrapper.
- `workflow/` — example scripts, useful when learning by example
  outside of skylab.

## Common tasks

- **Run the MPAS-JEDI reference tests** — `ctest -R mpasjedi_` in the
  build tree (data fixture comes from `mpas-jedi-data`).
- **Run a 3DEnVar experiment** — driven by skylab + ewok; executable
  is `mpasjedi_variational.x` (or similar) invoked with a YAML.
- **Switch to single precision** — pass
  `-DDOUBLE_PRECISION=OFF` (note this needs the matching MPAS-Model
  build).
- **Add a new variable change** — drop a class under
  `src/mpasjedi/VariableChange/`, register, add a test YAML.

## Gotchas

- `mpas-jedi` requires the bundle's `mpas` repo built with the right
  core(s). The 2-stream I/O test data (current `develop`) was generated
  with **single-precision** MPAS; the test `.ref` files match that run.
  If you build MPAS with double precision, ctests will fail on numeric
  mismatches even though the code is correct — both repos must use the
  same precision.
- The standalone tarball fallback for test data is pinned to
  `3.1.0.jcsda` in `mpas-jedi/CMakeLists.txt` — bumping mpas-jedi may
  require also bumping the tarball pin.
- The `workflow/` csh scripts are *legacy* — for production
  experiments use skylab + ewok instead.
- The `3denvar_2stream_bumploc` test was removed in develop (2026-04)
  as part of the 2-stream I/O migration. If your branch still has it,
  delete `test/testinput/3denvar_2stream_bumploc.yaml` and its `.ref`.
- **New `"regional nn fill distance in km"` geometry parameter
  (2026-05):** added in `src/mpasjedi/Geometry/GeometryParameters.h`
  and the Fortran geom module (mpas-jedi#1199). Controls nearest-
  neighbour fill distance for regional mesh geometries.
- **Regional mesh ctests (2026-05):** two new ctests added:
  `3denvar_multi_resolution_regional` (384 km mesh) and
  `parameters_bumploc_regional` (480 km mesh with ensemble). Requires
  the new `mpas-jedi-data` regional test data (384km/480km NetCDF +
  ensemble members). Test namelists live in
  `test/namelists/384km/` and `test/namelists/480km/`.
- **`smoke_fine` added to `field_is_scalar` logic (2026-05):**
  `src/mpasjedi/Fields/mpas_fields_mod.F90` / the DA path — affects
  which fields are treated as scalars. If you have branches patching this
  function, check for merge conflicts (mpas-jedi#1136).
- **2m T,q now updated between outer loops (2026-05):**
  `src/mpasjedi/State/mpas_state_mod.F90` (mpas-jedi#1174). Affects
  multi-outer-loop experiments; reference outputs were updated.
- **EnKF test references updated (2026-05):** three enkf-related ctest
  outputs were updated to reflect the GETKF QC change from oops (apply
  QC to ensemble mean/modulated members). Rebuild and re-run ctests if
  your build predates this.
- **CO2 absorber for CrIS & IASI (2026-06, mpas-jedi#1203):** the CRTM
  operator now includes CO2 as an absorber for CrIS and IASI channels.
  Affects HofX and radiance assimilation tests — `hofx3d.ref`, `hofx4d.ref`,
  and related refs were updated.
- **CDA ctest expansion (2026-06, mpas-jedi#1146):** `3dfgat_cda.yaml`,
  `4dfgat_cda.yaml`, and associated ref files were updated to exercise new
  CDA capabilities. `3dfgat_cda` was also given `tier2` labels (mpas-jedi#1205).
- **Integer field interpolation type (2026-06, mpas-jedi#1200):** the
  interpolation type for integer fields was corrected in
  `src/mpasjedi/Fields/mpas_fields_mod.F90`. Affects model and state
  init paths.
- **Broad ref-output update (2026-06, mpas-jedi#1204):** 55 files changed
  (2687 ins, 2637 del) — letkf, lgetkf, parameters_bumpcov/bumploc,
  process_perts_spectral, rtpp, 3denvar, 4denvar, eda, forecast, hofx,
  dirac, gen_ens_pert_B refs all regenerated. Any branch with local edits
  to these files should expect heavy merge conflicts.

## Further reading

- jedi-docs.jcsda.org → JEDI Components → MPAS-JEDI.
- Related briefs: `jedi-knowledge/oops.md`, `jedi-knowledge/ufo.md`,
  `jedi-knowledge/saber.md`, `jedi-knowledge/vader.md`,
  `jedi-knowledge/crtm.md`, `jedi-knowledge/mpas.md`,
  `jedi-knowledge/mpas-jedi-data.md`, `jedi-knowledge/jedi-bundle.md`.
