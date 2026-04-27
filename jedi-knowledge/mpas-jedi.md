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
  core(s); single-precision MPAS is incompatible with default
  double-precision mpas-jedi unless both are toggled together.
- The standalone tarball fallback for test data is pinned to
  `3.1.0.jcsda` in `mpas-jedi/CMakeLists.txt` — bumping mpas-jedi may
  require also bumping the tarball pin.
- The `workflow/` csh scripts are *legacy* — for production
  experiments use skylab + ewok instead.

## Further reading

- jedi-docs.jcsda.org → JEDI Components → MPAS-JEDI.
- Related briefs: `jedi-knowledge/oops.md`, `jedi-knowledge/ufo.md`,
  `jedi-knowledge/saber.md`, `jedi-knowledge/vader.md`,
  `jedi-knowledge/crtm.md`, `jedi-knowledge/mpas.md`,
  `jedi-knowledge/mpas-jedi-data.md`, `jedi-knowledge/jedi-bundle.md`.
