# crtm

**Repository:** https://github.com/jcsda/CRTMv3
**Branch tracked by bundle:** develop
**Role in JEDI:** Community Radiative Transfer Model — the satellite radiance forward operator (and its tangent linear / adjoint / K-matrix) used by UFO's radiance observation operators for IR, MW, and visible sensors.

## What it is

CRTM is a Fortran library that computes top-of-atmosphere brightness
temperatures / radiances from atmospheric and surface state, plus their
TL/AD/K-matrix derivatives, for many satellite sensors (AMSU-A/B, ATMS,
CrIS, IASI, AIRS, ABI, AHI, GMI, SSMIS, ...). It is sensor-coefficient
driven: per-sensor SpcCoeff, TauCoeff, AerosolCoeff, CloudCoeff, EmisCoeff
binary files supply the radiative properties; the library code is
sensor-agnostic.

CRTM is developed independently of JEDI (originally JCSDA's standalone
radiative-transfer library), and JEDI consumes it as an external Fortran
dependency. UFO's `RTTOV` and `CRTM` observation operators are alternative
radiance forward models; this repo provides the CRTM side. Current
version on develop: **3.1.3** (`VERSION.cmake`; REL-3.1.3 added IFX
compiler support).

## How it fits into the bundle

- **Depends on:** Fortran 2008 compiler, NetCDF (Fortran, REQUIRED),
  optional OpenMP. No JEDI deps — CRTM does not link against
  oops/eckit/atlas.
- **Depended on by:** `ufo` (when built with CRTM enabled). UFO uses
  `find_package(crtm 2.4.x)` (and special-cases
  `crtm_VERSION >= 3.0.0`), linking its CRTM observation operator
  against `libcrtm`.
- **Build position:** can be built early; has no JEDI prerequisites. In
  the bundle CRTM builds before UFO.

## Key directories

- `src/` — Fortran sources of the CRTM library, organized by physics
  module.
  - Top-level files: `CRTM_Module.F90` (umbrella module),
    `CRTM_LifeCycle.f90`, `CRTM_Forward_Module.f90`,
    `CRTM_Tangent_Linear_Module.f90`, `CRTM_Adjoint_Module.f90`,
    `CRTM_K_Matrix_Module.f90`, `CRTM_Parameters.f90`.
  - `Atmosphere/` — atmospheric state, hypsometric, RH, Cloud, Aerosol.
  - `Surface/`, `SfcOptics/`, `SensorInfo/`, `InstrumentInfo/`,
    `ChannelInfo/`, `GeometryInfo/`, `Options/`, `Ancillary/`.
  - `AtmAbsorption/`, `AtmOptics/`, `AtmScatter/`, `Source_Functions/`,
    `RTSolution/`, `Interpolation/`.
  - `Coefficients/` — coefficient I/O modules (`CRTM_SpcCoeff.f90`,
    `CRTM_TauCoeff.f90`, `CRTM_AerosolCoeff.f90`,
    `CRTM_CloudCoeff.f90`, plus per-sensor IR/MW/VIS variants and
    `ACCoeff/`, `BeCoeff/`, `CloudCoeff/`, `EmisCoeff/`, `FitCoeff/`,
    `NLTECoeff/`, `SpcCoeff/`, `TauCoeff/` subdirs).
  - `Zeeman/`, `NLTE/`, `AntennaCorrection/`, `Statistics/`,
    `CRTM_Utility/`, `TauProd/`, `TauRegress/`, `Test_Utility/`,
    `Utility/`, `User_Code/`.
- `test/` — ctest harness.
  - `test/mains/application/` — `check_crtm.F90`,
    `check_crtm_random_profiles.F90`, `check_tropics.f90`.
  - `test/mains/regression/` — `forward/`, `tangent_linear/`,
    `adjoint/`, `k_matrix/` regression tests (+ shared `incsrc/`).
  - `test/mains/unit/` — unit tests (`Unit_Test/`, `input_output/`).
  - `test/crtm_data_downloader.py` — pulls coefficient binaries.
  - `test/CMakeLists.txt` — coefficient download/symlink logic; exports
    the global CMake property `CRTM_TESTFILES_PATH` so downstream
    projects (UFO) can find the downloaded coefficients.
- `cmake/` — compiler flag files per compiler.
- `CRTM_V30_TEST/` — legacy v3.0-era test sources kept at top level;
  not part of the CMake test suite.
- `Get_CRTM_Binary_Files.sh` — downloads the `fix/` coefficient tarball
  from UCAR's GDEX (legacy / standalone use).
- `Set_CRTM_Environment.sh` — env helpers for standalone builds.

## Key entry points

- `src/CRTM_Module.F90` — the public umbrella module; `use CRTM_Module`
  from a Fortran caller exposes Forward/TL/AD/K-matrix.
- `src/CRTM_Forward_Module.f90` — non-linear forward model entry
  (`CRTM_Forward`).
- `src/CRTM_Tangent_Linear_Module.f90`, `CRTM_Adjoint_Module.f90`,
  `CRTM_K_Matrix_Module.f90` — linearizations.
- `src/CRTM_LifeCycle.f90` — `CRTM_Init` / `CRTM_Destroy`; coefficient
  loading happens here.
- `src/Atmosphere/CRTM_Atmosphere_Define.f90` — the
  `CRTM_Atmosphere_type` derived type, the primary input data structure.
- `test/mains/application/check_crtm.F90` — minimal example of using
  CRTM in a standalone program; the easiest place to learn the call
  sequence.
- `README_JEDI.md` — JEDI-specific build instructions.

## Common tasks

- **Build standalone for testing** — see `README_JEDI.md`.
  `mkdir build && cd build && cmake .. && make -j8 && ctest -j8`. As of
  v3.1.0 the test coefficient tarball is auto-downloaded from GDEX
  during configure.
- **Build inside the JEDI bundle** — built automatically; the `crtm`
  target is consumed by UFO via `find_package(crtm)`.
- **Override the coefficient location** — set
  `-DFIX_FILE_PATH=<path>` at configure (defaults to
  `$LOCAL_PATH_JEDI_TESTFILES/fix` if that env var is set, else
  `<src>/fix`). If the path doesn't exist, configure downloads the
  `fix_REL-3.1.2.0` tarball into `<src>/test-data-release/` (or uses the
  `CRTM_BINARY_FILES_TARBALL` env var if set). See `test/CMakeLists.txt`.
- **Add or update a sensor** — drop new SpcCoeff / TauCoeff / EmisCoeff
  binaries into the coefficient dir; CRTM is sensor-data driven and
  library code does not need changes for new sensors using existing
  physics.
- **Run only TL/AD checks** — `ctest -R tangent_linear` /
  `ctest -R adjoint` / `ctest -R k_matrix`.
- **Get coefficient files outside JEDI** — `./Get_CRTM_Binary_Files.sh`.

## Gotchas

- The bundle's CRTM CMake (`CMakeLists.txt`) overrides
  `CMAKE_INSTALL_PREFIX` to `${CMAKE_BINARY_DIR}/lib` when the user did
  not set one — intentional but surprising; CRTM ends up under the build
  tree rather than a system prefix unless you pass
  `-DCMAKE_INSTALL_PREFIX`.
- Coefficient files are large and version-keyed to the library —
  mismatched coefficient sets vs library version will manifest as
  runtime read errors. The default `Get_CRTM_Binary_Files.sh` pulls the
  matching set; do not mix coefficients across major versions.
- CRTM is CC0 / Public Domain (`LICENSE` / `COPYING`) — different
  licensing from the JEDI Apache-2.0 stack; keep that in mind for
  redistribution.
- CRTM is **not** templated on OOPS interfaces; UFO does the OOPS-side
  adaptation. Work to "make CRTM JEDI-aware" should be done in UFO, not
  here.
- Builds are tested primarily on gfortran; Intel Fortran works but
  receives less attention.
- The repository contains both the modern CMake build and a legacy
  autotools / `Makefile` build under `src/`. JEDI uses CMake exclusively;
  ignore the `Makefile` artifacts.
- **Coefficient download path changed (2026-06, crtm#302):** The directory
  where CRTM downloads its coefficient files was decoupled from UFO's
  expected path. `test/CMakeLists.txt` now sets the global CMake property
  `CRTM_TESTFILES_PATH` so UFO can discover the downloaded coefficient path
  without hard-coding the UFO subdirectory (paired with ufo#4123). If you
  have custom logic that assumed coefficients land under the UFO subtree,
  update it to read this property instead.
- **Unit-test tag selector added (2026-05, crtm#301):** unit tests carry a
  ctest label/tag selector; `test_check_crtm_random` was removed from the
  default CI tier ("tier2"-labeled) to cut CI time.

## Further reading

- In-repo: `README.md`, `README_JEDI.md`, `NOTES`.
- crtm-support@googlegroups.com, https://forums.jcsda.org/.
- jedi-docs.jcsda.org: UFO observation-operator pages reference CRTM
  use.
- Related briefs: `jedi-knowledge/ufo.md`, `jedi-knowledge/rttov.md`,
  `jedi-knowledge/jedi-bundle.md`.
