# oops

**Repository:** https://github.com/jcsda-internal/oops
**Branch tracked by bundle:** develop
**Role in JEDI:** Foundational C++/Fortran framework providing the abstract data assimilation algorithms, interfaces, and toy models that every model-specific JEDI repo (fv3-jedi, mpas-jedi, soca, ufo, ioda, saber, vader, ...) plugs into.

## What it is

OOPS (Object Oriented Prediction System) is the core of JEDI. It defines the
templated C++ class hierarchy for states, increments, geometries, models,
observation operators, error covariances, minimizers, and full DA
applications (3DVar, 4DVar, EnVar, EnKF/LETKF/GETKF, hybrid). Concrete
models (e.g. fv3-jedi, mpas-jedi, soca) implement a `Traits` struct that
satisfies the OOPS interfaces; OOPS then composes them into runnable
applications. OOPS itself is model-agnostic.

OOPS originates from ECMWF's OOPS prototype and is co-developed with JCSDA.
It also ships two reference / toy models (Lorenz-95 and the QG
shallow-water model) used as the canonical regression and demonstration
platform for new algorithms before they are wired into operational models.

## How it fits into the bundle

- **Depended on by:** essentially everything else in the bundle — ufo,
  ioda, saber, vader, fv3-jedi, mpas-jedi, soca, coupling. They
  `find_package(oops 1.10.0 REQUIRED)`.
- **Depends on:** eckit, fckit, atlas, MPI, NetCDF (with parallel),
  Eigen3, Boost, LAPACK/MKL, optionally GPTL and nlohmann_json.
- **Build position:** built very early in the bundle (after
  eckit/fckit/atlas/jedicmake). See top-level `jedi-bundle/CMakeLists.txt`.

## Key directories

- `src/oops/` — the OOPS library proper (the `oops::` namespace).
  - `base/` — core abstract classes: `State`, `Increment`, `Model`,
    `Geometry`, `Variables`, `ObsSpaceBase`, `Locations`,
    `LinearModelBase`, etc.
  - `interface/` — templated interface wrappers (`Geometry<MODEL>`,
    `State<MODEL>`, `LinearObsOperator<OBS>`, ...) that adapt
    model-specific implementations to the generic algorithms.
  - `assimilation/` — cost functions (`CostFct3DVar`, `CostFct4DVar`,
    `CostFct4DEnsVar`, `CostFctFGAT`, `CostFctWeak`), minimizers (DRPCG,
    PCG, GMRES family, MINRES, Lanczos, saddle-point, LETKF/GETKF
    solvers).
  - `runs/` — top-level `Application` classes: `Variational`, `HofX3D`,
    `HofX4D`, `Forecast`, `EnsembleApplication`, `LocalEnsembleDA`,
    `ConvertState`, `EnsMeanAndVariance`, `GenEnsPertB`, `Test`. These
    are what model executables instantiate.
  - `generic/` — model-agnostic implementations: `PseudoModel`,
    `IdentityModel`, `HybridLinearModel`, `Diffusion`,
    `UnstructuredInterpolator`, `AtlasInterpolator`,
    `GlobalInterpolator`, `gc99`/`soar` correlation functions, FFT
    helpers.
  - `coupled/` — generic coupled-model infrastructure
    (`GeometryCoupled`, `ModelCoupled`, `StateCoupled`, `TraitCoupled`,
    ...).
  - `util/` — `DateTime`, `Duration`, `Logger`, `Printable`, `Factory`,
    `Parameters`/JSON-schema infrastructure, `AtlasArrayUtil`, FieldSet
    helpers, `Random`, `abor1_ftn`.
  - `mpi/` — MPI scope/communicator helpers.
  - `atlas/` — thin Atlas interpolator wrapper.
  - `contrib/` — DCMIP test initial conditions.
- `l95/` — Lorenz-95 toy model (`l95/src/lorenz95/`,
  `l95/src/executables/`, `l95/test/`).
- `qg/` — Quasi-Geostrophic shallow-water toy model. `qg/model/`,
  `qg/mains/`, `qg/test/`.
- `coupled/` — tests for the L95+QG coupled toy at the bundle level.
- `share/oops/` — installed shared assets (e.g. `suppressions/` for
  valgrind/sanitizers).
- `tools/` — `cpplint.py`, `compare.py` (test ref comparison),
  `gen_yaml_ensemble.sh`, `plot.py`, `test_wrapper.sh`.
- `docs/` — Doxygen configuration; built only with `-DENABLE_OOPS_DOC=ON`.

## Key entry points

- `src/oops/runs/Run.cc` / `Run.h` — the harness every JEDI executable
  starts with (`oops::Run run(argc, argv); return run.execute(application);`).
- `src/oops/runs/Variational.h` — variational DA driver (3DVar, 4DVar,
  EnVar, hybrid).
- `src/oops/runs/HofX4D.h`, `HofX3D.h` — observation forward operator
  applications.
- `src/oops/runs/LocalEnsembleDA.h` — EnKF / LETKF / GETKF entry point.
- `src/oops/assimilation/CostFct4DVar.h`, `CostFct3DVar.h`,
  `CostFct4DEnsVar.h` — cost-function definitions.
- `src/oops/base/Variables.h`, `base/Variable.h` — variable-naming
  abstraction every model must speak.
- `src/oops/util/parameters/` — `Parameters` / JSON-schema-aware YAML
  configuration system; understanding this unlocks every JEDI yaml.
- `qg/mains/qg4DVar.cc` — canonical example of how a model wires its
  `Traits` into `Variational<QG>`.

## Common tasks

- **Run the QG 4DVar reference test** — configure with
  `-DENABLE_QG_MODEL=ON` (default), then `ctest -R qg_4dvar`.
- **Run the L95 reference tests** — `-DENABLE_LORENZ95_MODEL=ON`
  (default), `ctest -R l95_`.
- **Add a new DA application** — new header in `src/oops/runs/` deriving
  from `oops::Application`, register in `src/oops/runs/CMakeLists.txt`,
  then instantiate from the model's `mains/` (template:
  `qg/mains/qg4DVar.cc`).
- **Add a new minimizer** — drop a header in `src/oops/assimilation/`
  following `DRPCGMinimizer.h` / `PCGMinimizer.h`, register it in
  `Minimizer.h`'s factory.
- **Wire YAML <-> code** — use `oops::Parameters` (see
  `src/oops/util/parameters/`); JSON-schema is auto-emitted via
  `cmake/oops_output_json_schema.cmake`.
- **Time/date arithmetic** — `oops::DateTime` and `oops::Duration`
  (`src/oops/util/DateTime.h`, `Duration.h`).
- **Logging** — `oops::Log::info()/debug()/test()/trace()` from
  `src/oops/util/Logger.h`.

## Gotchas

- OOPS requires `NETCDF4_PARALLEL` — a serial-only NetCDF build fails
  configure with `Missing PARALLEL feature for NetCDF`.
- Heavy template use; compile errors can be huge. Build the toy models
  first to validate the stack before iterating on a model interface.
- `oops::Variables` / `Variable` API is evolving (stricter typing in
  1.10.x); old downstream code using `Variables(std::vector<std::string>)`
  may need updates.
- Downstream pins `find_package(oops 1.10.0 REQUIRED)`; bumping
  `project(... VERSION ...)` here means bumping pins everywhere.
- The top-level `coupled/` subdirectory builds only when both QG and L95
  are enabled.

## Further reading

- In-repo: `README.md`, `docs/Doxyfile.in`.
- jedi-docs.jcsda.org: "OOPS" section under JEDI Components.
- Related briefs: `jedi-knowledge/ufo.md`, `jedi-knowledge/ioda.md`,
  `jedi-knowledge/saber.md`, `jedi-knowledge/vader.md`,
  `jedi-knowledge/fv3-jedi.md`, `jedi-knowledge/mpas-jedi.md`,
  `jedi-knowledge/soca.md`, `jedi-knowledge/coupling.md`,
  `jedi-knowledge/jedi-bundle.md`.
