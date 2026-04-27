# saber

**Repository:** https://github.com/jcsda-internal/saber
**Branch tracked by bundle:** develop
**Role in JEDI:** Background-error covariance (B-matrix) toolkit for JEDI. Implements the parametric, ensemble, and hybrid B's used by every variational/EnVar application — BUMP, spectral B (SPECTRALB), FastLAM, diffusion, GSI-bec interface, plus block-chain machinery to compose them.

## What it is

SABER (System-Agnostic Background Error Representation) provides a library
of "saber blocks" — outer transforms (e.g. spectral-to-grid, vertical
balance, ψχ→u,v, variable change via VADER) and central blocks (e.g.
parametric correlations, ensemble localization) — that compose into a full
B operator. JEDI's `ErrorCovariance` and `Localization` factories are
populated from SABER, and downstream models pull the resulting B
implementations through OOPS.

SABER bundles a self-contained toy model called QUENCH (in `quench/`) that
mirrors the OOPS `Traits` interface and is used to exercise B
configurations end-to-end (covariance training, randomization, dirac
response) without needing a full atmospheric model.

## How it fits into the bundle

- **Depends on:** `oops 1.10.0`, `vader 1.7.0`, atlas, eckit, fckit,
  NetCDF, MPI, LAPACK/MKL. Optional: `ECTRANS`, `FFTW`, `ip` (NCEP),
  `gsibec` (GSI background error), Torch (for the `torchbalance` block).
- **Depended on by:** every JEDI model that uses a non-trivial B —
  fv3-jedi, mpas-jedi, soca, qg(coupled). They `find_package(saber ...)`
  and link against the appropriate saber libraries.
- **Build position:** after oops and vader.

## Key directories

- `src/saber/` — the saber library (split per block family).
  - `oops/` — glue to OOPS: `ErrorCovariance.h`, `Localization.h`,
    `ErrorCovarianceParameters.h`, `ErrorCovarianceToolbox.h`,
    `ProcessPerts.h`, plus the `instantiateCovarFactory.h` /
    `instantiateLocalizationFactory.h` that downstream code calls to
    register blocks.
  - `blocks/` — the block-chain framework: `SaberOuterBlockBase`,
    `SaberCentralBlockBase`, `SaberOuterBlockChain`,
    `SaberEnsembleBlockChain`, `SaberHybridBlockChain`,
    `SaberParametricBlockChain`, `instantiateBlockChainFactory.h`.
  - `bump/` — the BUMP library (Background error on an Unstructured
    Mesh, Benjamin Ménétrier). Core blocks: `NICAS` (correlation),
    `StdDev`, `VerticalBalance`, `PsiChiToUV`, `NICASFilter`. Heavy use
    of fypp templating (`*.fypp`).
  - `spectralb/` — spectral B-matrix blocks: `SpectralCovariance`,
    `SpectralCorrelation`, `SqrtOfSpectralCorrelation`,
    `SqrtOfSpectralCovariance`, `SpectralToGauss`, `SpectralToSpectral`,
    `SpectralAnalyticalCorrelation`, `GaussUVToGP`, `HydrostaticPressure`.
  - `fastlam/` — FastLAM (limited-area model fast B), with halo and
    spectral layer variants (requires FFTW or ECTRANS).
  - `bifourier/` — bi-Fourier transforms for LAM grids.
  - `diffusion/` — diffusion-based correlations (`Diffusion`,
    `DiffusionFilter`, `DiffusionImpl`).
  - `interpolation/` — Atlas interp wrappers, `GaussToCS` (Gaussian to
    cubed-sphere), `Rescaling`, `VertProj`, `VectorFieldMetadata`.
  - `gsi/` — GSI-bec interface (`GSIBlockChain`, `covariance/`, `grid/`,
    `utils/`); requires `gsibec`.
  - `vader/` — saber blocks that wrap VADER variable changes.
  - `coupled/` — `CoupledErrorCovariance` for multi-component coupled DA.
  - `generic/` — `ID` (identity), `StdDev`, `DuplicateVariables`,
    `OrographicInterp`, `ShadowLevels`, `VertLoc`, `VertLocInterp`,
    `WriteFields`.
  - `torchbalance/` — Torch-based learned balance operator (optional,
    requires Torch_ROOT).
  - `util/` — calibration, binned-fieldset helpers, NetCDF/coordinate
    Fortran helpers.
- `quench/` — toy model.
  - `quench/src/` — `Geometry`, `State`, `Increment`, `Fields`,
    `Covariance`, `VariableChange`, `LinearVariableChange`,
    `Interpolation`, `Traits.h`.
  - `quench/mains/` — `quenchErrorCovarianceToolbox.cc`,
    `quenchConvertState.cc`, `quenchProcessPerts.cc`,
    `quenchSubCommErrorCovarianceToolbox.cc`,
    `quenchCoupledErrorCovarianceToolbox.cc`.
- `test/` — tests with per-block YAML inputs.
  - `test/testinput/` — hundreds of YAMLs (`dirac_*`, `convertcov_*`,
    `compare_diagnostics_*`).
  - `test/testlist/` — per-feature CMake test lists (`saber_bump.cmake`,
    `saber_spectralb.cmake`, `saber_fastlam.cmake`, `saber_gsi-*.cmake`,
    `saber_torchbalance.cmake`, ...).
  - `test/testref/` — reference outputs.
  - `test/fctest/` — Fortran-side tests.
- `tools/` — `saber_plot`, `saber_plot.py`,
  `saber_correlation_variance.py`, `saber_compare_dirac_diagnostics.py`,
  `saber_fit_function.py`, `saber_tidy_yamls.py`, `saber_gcov.sh`,
  `saber_vtune.sh`, `saber_cpplint.py`.
- `docs/` — Doxygen + `tutorial/`, `bump/`, `yaml/` (curated example
  YAMLs), `history/`.

## Key entry points

- `src/saber/oops/ErrorCovariance.h` — the OOPS-facing
  `ErrorCovariance<MODEL>` implementation. The thread that ties saber
  blocks into a JEDI Variational application.
- `src/saber/oops/instantiateCovarFactory.h` — the registration call
  models add to register all SABER blocks with OOPS.
- `src/saber/blocks/SaberOuterBlockBase.h`, `SaberCentralBlockBase.h` —
  base classes for adding a new block.
- `src/saber/blocks/SaberOuterBlockChain.cc`,
  `SaberParametricBlockChain.cc`, `SaberHybridBlockChain.cc` — how
  blocks compose into a full B.
- `src/saber/bump/BUMP.h`, `NICAS.h`, `StdDev.h` — the most widely used
  parametric covariance blocks.
- `quench/mains/quenchErrorCovarianceToolbox.cc` — the canonical
  executable for B training/testing without a real model.
- `docs/yaml/error_covariance_training_tutorial_*.yaml` — annotated
  tutorial YAMLs.

## Common tasks

- **Run tests for one block family** — `ctest -L saber -R bump`, or
  `ctest -R saber_spectralb_`, etc. Test sets are gated on detected
  optional deps (FFTW, ECTRANS, gsibec, Torch).
- **Train a B with QUENCH** — build the QUENCH executables
  (`-DENABLE_QUENCH=ON`, default), then run
  `quenchErrorCovarianceToolbox <yaml>`. Tutorial YAMLs are in
  `docs/yaml/`.
- **Add a new outer block** — derive from `SaberOuterBlockBase`,
  register via `SABER_OUTER_BLOCK_REGISTRY` macro, drop sources into a
  subdir of `src/saber/`, list them in that subdir's `CMakeLists.txt`.
- **Add a new central block** — same pattern with
  `SaberCentralBlockBase`.
- **Plot dirac responses** — `tools/saber_plot.py` /
  `tools/saber_plot/`.
- **Check covariance vs reference** —
  `tools/saber_compare_dirac_diagnostics.py`.
- **Run cpplint** — `ctest -R saber_coding_norms_src`.

## Gotchas

- Many block families are conditional on optional deps. Check the
  configure output for "SABER block X is enabled / NOT enabled" lines
  (printed in top-level `CMakeLists.txt`).
- BUMP uses `fypp` preprocessing (`*.fypp`); editing those requires
  `fypp` and a re-cmake. Generated `.F90` files live in the build tree.
- Atlas version gates several features: `ATLAS_REGIONAL_INTERP` for
  Atlas > 0.38.1, `ATLAS_MAKE_SPARSE` for > 0.41, spherical-vector
  interp meta for ≥ 0.44, `ATLAS_SCOPE_ISSUE_RESOLVED` for ≥ 0.46.
  Surprises with mismatched Atlas versions usually trace back here.
- Tests require `jedi-model-data` (>= 1.0.0); without it the `test/`
  subdirectory is skipped silently.
- `quench` is build-on-by-default — turn off with `-DENABLE_QUENCH=OFF`
  if you don't have jedicmake or want a faster build.
- BUMP instrumentation is a separate compile flag
  (`-DENABLE_BUMP_INSTRUMENTATION=ON`) used for performance studies.

## Further reading

- In-repo: `README.md`, `docs/tutorial/`, `docs/yaml/` (annotated YAMLs),
  `docs/bump/plot.py`.
- jedi-docs.jcsda.org: SABER section, plus the BUMP and SPECTRALB
  tutorials.
- Related briefs: `jedi-knowledge/oops.md`, `jedi-knowledge/vader.md`,
  `jedi-knowledge/fv3-jedi.md`, `jedi-knowledge/mpas-jedi.md`,
  `jedi-knowledge/soca.md`, `jedi-knowledge/jedi-model-data.md`,
  `jedi-knowledge/coupling.md`.
