# ufo

**Repository:** https://github.com/jcsda-internal/ufo
**Branch tracked by bundle:** develop
**Role in JEDI:** Unified Forward Operator — the observation-space side of JEDI. Houses every observation operator (the forward / TL / AD operations from model state to simulated obs), every QC filter, bias-correction predictors, and the OBS Traits adapter that lets oops-side algorithms drive observations.

## What it is

UFO provides JEDI's `ObsTraits` and the bulk of observation-space code:

- **Forward operators** (`operators/`) — radiance (CRTM, RTTOV), GNSS-RO,
  ground-based GNSS, sondes/aircraft/ATMS/AMSU-A/IASI/CrIS/AIRS/AHI/ABI/
  scatwind/SST/aerosol/AOD/ozone/cloud-top/etc.
- **QC filters** (`filters/`) — gross checks, background checks, bias
  predictors, history of value rejections.
- **Obs functions** (`filters/obsfunctions/`) — includes (2026-05)
  `CircularDifference` (angular difference for wind directions),
  `TimeBinner` (time-window binning), `ProfileVerticalSmoothing`
  (vertical smoothing along a profile), and `Statistic` (global
  statistics across all MPI ranks).
- **Bias correction** (`predictors/` + `ObsBias*`) — the variational
  bias-correction infrastructure.
- **Variable transforms** (`variabletransforms/`) — observation-side
  variable conversions parallel to VADER's model-side ones.
- **Profile QC** (`profile/`), **sampled locations**
  (`SampledLocations*`), **error models** (`errors/`), **field-of-view**
  (`fov/`), **super-obing** (`superob/`).

This is the largest and most actively-developed JEDI repo by file count.

## How it fits into the bundle

- **Depends on:** `oops`, `ioda`, `crtm` (and/or `rttov` if
  `BUILD_RTTOV=ON`), atlas, eckit, fckit. Optional: `ropp-ufo` for
  GNSS-RO via ROPP.
- **Depended on by:** every model wrapper (fv3-jedi, mpas-jedi, soca,
  coupling) that runs DA against observations, plus `simobs`.
- **Build position:** after oops + ioda + crtm.

## Key directories

- `src/ufo/` — the UFO library.
  - Top-level: `ObsTraits.h`, `ObsOperator.{h,cc}`,
    `LinearObsOperator.{h,cc}`, `GeoVaLs.{h,cc}`, `ObsFilters.{h,cc}`,
    `ObsBias*`, `ObsDiagnostics.{h,cc}`, `SampledLocations.{h,cc}`.
  - `operators/` — forward operators, organized by instrument /
    family (e.g. `radiosonde/`, `aircraft/`, `crtm/`, `rttov/`,
    `gnssro/`, `sfc/`, `atmsfclnda/`, …).
  - `filters/` — QC filters (Gaussian thinning, gross check, bias-
    threshold, derivative checks, history, …).
  - `predictors/` — bias-correction predictors (constant,
    legendre/laguerre, lapse-rate, satellite scan position, etc.).
  - `variabletransforms/` — observation-space variable conversions.
  - `obslocalization/` — observation localization for EnKF/LETKF.
  - `errors/` — observation-error models.
  - `profile/` — vertical-profile QC machinery.
  - `superob/` — observation thinning / super-obing.
  - `fov/` — field-of-view geometry.
  - `basis/`, `utils/` — supporting infrastructure.
  - `instantiateObsFilterFactory.h`, `instantiateObsLocFactory.h` —
    factory registration callers.
- `test/` — extensive test suite. `test/testinput/` includes
  `instrumentTests/` and `unit_tests/` subtrees.
- `tools/` — helper scripts.
- `docs/` — design notes, including `docs/organization.md` (read this
  for an overview of UFO's internal layout).
- `resources/` — packaged config/resource files.

## Key entry points

- `src/ufo/ObsTraits.h` — the `UfoTraits` struct that defines the OBS
  Traits implementation.
- `src/ufo/ObsOperator.{h,cc}` — the OOPS-facing
  `oops::ObsOperatorBase` implementation.
- `src/ufo/LinearObsOperator.{h,cc}` — TL/AD wrapper.
- `src/ufo/GeoVaLs.{h,cc}` — model values interpolated to observation
  locations (the bridge between model and obs space).
- `src/ufo/ObsFilters.{h,cc}` — pipeline of QC filters.
- `src/ufo/ObsBias*` — variational bias correction.
- `src/ufo/operators/<family>/` — pick one to learn the operator
  pattern (e.g. `operators/identity/` is the simplest).
- `docs/organization.md` — narrative orientation to the code layout.

## Common tasks

- **Run the UFO test suite** — `ctest -L ufo` in the build tree.
- **Add a new observation operator** —
  1. Create `src/ufo/operators/<family>/<MyOp>.{h,cc}` deriving from
     `ObsOperatorBase` (and a TL/AD partner from
     `LinearObsOperatorBase`).
  2. Register in `src/ufo/operators/<family>/CMakeLists.txt` and
     `instantiateObsOperatorFactory.h`.
  3. Add a test YAML and reference output under `test/testinput/` and
     `test/testref/`.
- **Add a new QC filter** — derive from `ObsFilterBase`, register via
  `instantiateObsFilterFactory.h`, add a test.
- **Use CRTM vs RTTOV** — selected via the YAML
  `obs operators[i].name` field (`CRTM` vs `RTTOV`); both backends live
  side-by-side under `operators/`.

## Gotchas

- Operator and filter factories register via global initializers — if
  a new file isn't picked up at runtime, double-check the
  `instantiate*Factory.h` includes.
- `GeoVaLs` variable names must match what VADER (model side) and
  the operator (obs side) agree on — name drift is a common source of
  silent zero outputs.
- The CRTM and RTTOV operators both require their respective
  coefficient files at runtime; missing or version-mismatched
  coefficients produce confusing errors.
- The repo is large; expect non-trivial build times. `ccache` helps.
- `ImpactHeightCheck.h` — the superrefraction flag was renamed/changed
  in develop (2026-04). If you have a local branch touching GNSS-RO,
  check for conflicts with this change.
- `CloudDetectMinResidualIR.cc` had ~15 lines removed in the same round
  of warning cleanup. If merging from an older state, expect a conflict
  in that file.
- **QC flag API change (2026-05):** The QC flag `ObsDataVector` is now
  passed as a reference rather than a shared pointer throughout the filter
  stack (ufo#4112). All ~150 filter classes were updated; any local
  branches touching filters will need rebasing.
- **gsw cmake fix (2026-05):** `gsw_LIBRARIES` cmake usage was corrected
  (ufo#4135); if your build was working around this, the workaround may
  now conflict.
- **GNSSRO test data downsized (2026-05):** large GNSSRO geoval files
  were replaced with smaller variants in ufo-data (ufo-data#562). Tests
  that referenced the old file sizes will need updating.
- **New `DuplicateThinning` filter (2026-05):**
  `src/ufo/filters/DuplicateThinning.{h,cc}` (ufo#4086). Registered in
  `instantiateObsFilterFactory.h`. Use in YAML as
  `filter: Duplicate Thinning`.
- **New `ProfileVerticalSmoothing` obs function (2026-05):**
  `src/ufo/filters/obsfunctions/ProfileVerticalSmoothing.{h,cc}`
  (ufo#4032). Documented in jedi-docs at
  `docs/jedi-components/ufo/qcfilters/obsfunctions/ProfileVerticalSmoothing.rst`.
- **New `Statistic` obs function (2026-05):**
  `src/ufo/filters/obsfunctions/Statistic.{h,cc}` (ufo#4091). Computes
  global statistics (mean, RMS, …) across all MPI ranks.
- **`ObsErrorDiffusion` mesh moved to `update` method (2026-05):**
  Mesh construction previously done at construction is now deferred to
  the `update` call (ufo#4129). Local branches overriding
  `ObsErrorDiffusion` will need rebasing.
- **`ObsSpace::put_db` now requires `dimList` (2026-05):** all call
  sites in ufo were updated (ufo#4126); check local branches that call
  `put_db` directly.

## Further reading

- In-repo: `README.md`, `docs/organization.md`.
- jedi-docs.jcsda.org → JEDI Components → UFO.
- Related briefs: `jedi-knowledge/oops.md`, `jedi-knowledge/ioda.md`,
  `jedi-knowledge/crtm.md`, `jedi-knowledge/rttov.md`,
  `jedi-knowledge/ropp-ufo.md`, `jedi-knowledge/ufo-data.md`,
  `jedi-knowledge/iodaconv.md`, `jedi-knowledge/jedi-bundle.md`.
