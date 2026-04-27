# gsw

**Repository:** https://github.com/jcsda-internal/GSW-Fortran
**Branch tracked by bundle:** develop
**Role in JEDI:** Fortran build of the Gibbs SeaWater (TEOS-10) Oceanographic Toolbox. Provides the seawater thermodynamics functions (conservative temperature, absolute salinity, density, freezing point, ...) used by ocean DA — primarily by VADER's marine recipes and SOCA.

## What it is

GSW is the JCSDA-Internal fork of the upstream TEOS-10 GSW-Fortran toolbox
(https://www.teos-10.org), repackaged with an ecbuild/CMake build so it
can be consumed as a CMake package by JEDI repos. The toolbox is the
standard implementation of TEOS-10 (the 2010 thermodynamic equation of
state of seawater): given practical salinity / in-situ temperature /
pressure it computes Conservative Temperature, Absolute Salinity, density,
sound speed, freezing temperatures, enthalpy, entropy, etc.

The science is unchanged from upstream; the JEDI fork supplies CMake
plumbing and module-export configuration so downstream JEDI code can
`find_package(gsw)` and `target_link_libraries(... gsw)`.

## How it fits into the bundle

- **Depends on:** Fortran compiler only (no other JEDI repos, no NetCDF
  link dep — though `gsw_data_v3_0.nc` is shipped under `test/` and used
  by the check program).
- **Depended on by:** `vader` (`find_package(gsw QUIET)`; if found,
  marine recipes — TEOS-10–based ocean variable conversions — are
  compiled in). Also used by `soca` and other ocean components.
- **Build position:** can be built very early; has no JEDI
  prerequisites.

## Key directories

- `modules/` — Fortran modules exporting parameter tables and constants.
  - `gsw_mod_kinds.f90`, `gsw_mod_teos10_constants.f90`,
    `gsw_mod_freezing_poly_coefficients.f90`,
    `gsw_mod_gibbs_ice_coefficients.f90`,
    `gsw_mod_specvol_coefficients.f90`, `gsw_mod_sp_coefficients.f90`,
    `gsw_mod_baltic_data.f90`,
    `gsw_mod_saar_data.f90` (Absolute Salinity Anomaly Ratio atlas),
    `gsw_mod_check_data.F90`, `gsw_mod_error_functions.f90`,
    `gsw_mod_toolbox.f90` (the umbrella module — `use gsw_mod_toolbox`
    to access the full API).
- `toolbox/` — one Fortran source per GSW function (~250 files:
  `gsw_ct_from_t.f90`, `gsw_rho.f90`, `gsw_sa_from_sp.f90`,
  `gsw_pt_from_ct.f90`, `gsw_freezing_*.f90`, `gsw_enthalpy_*.f90`, ...).
  Each is a thin numerical implementation of one TEOS-10 quantity.
- `test/` — verification harness.
  - `gsw_check_functions.f90` — runs every function against tabulated
    check values from `gsw_data_v3_0.nc`.
  - `gsw_poly_check.f90` — polynomial-approximation cross-checks.
  - `gsw_data_v3_0.nc` — the official TEOS-10 reference data (do not
    modify; the toolbox's correctness is validated against it).
  - `makefile`, `CMakeLists.txt`.
- `scripts/` — code-generation utilities used to regenerate the toolbox
  from MATLAB sources (most users do not run these).
- `cmake/` — compiler flags + ecbuild integration.
- `gsw-import.cmake.in` — package import file consumed by
  `find_package(gsw)`.

## Key entry points

- `modules/gsw_mod_toolbox.f90` — the public Fortran module umbrella;
  `use gsw_mod_toolbox, only: gsw_ct_from_t, gsw_rho, ...` from a JEDI
  caller.
- `toolbox/gsw_sa_from_sp.f90`, `gsw_ct_from_t.f90`, `gsw_rho.f90`,
  `gsw_pt_from_t.f90` — the most commonly called functions in JEDI
  ocean code.
- `test/gsw_check_functions.f90` — to learn the calling conventions and
  understand expected accuracy.
- `README` — upstream description (mentions running `gsw_check`).
- `README.JEDI.md` — JEDI build instructions.

## Common tasks

- **Build standalone** — `mkdir build && cd build && ecbuild
  --build=release <SRC_GSW> && make -j4` (per `README.JEDI.md`).
- **Run the check program** — after building, run the `gsw_check` test
  executable from the build tree's `test/` (it reads
  `gsw_data_v3_0.nc` and reports pass/fail per function). Typically
  wired in as a ctest.
- **Build inside a bundle** — automatic; `find_package(gsw)` in vader
  (and others) picks it up.
- **Wire VADER's marine recipes** — ensure GSW is in the build (look
  for `GSW FOUND; Including Marine Recipes gsw (...)` at vader's
  configure step). Otherwise vader silently skips marine recipes.
- **Add a new TEOS-10 function** — drop the new `gsw_*.f90` into
  `toolbox/`, add to `toolbox/CMakeLists.txt`, expose via
  `modules/gsw_mod_toolbox.f90` if appropriate. Prefer pulling from
  upstream TEOS-10 to keep numerical agreement.

## Gotchas

- This is a fork of an upstream third-party toolbox. Avoid hand-editing
  `toolbox/*.f90` for science changes — they should track upstream
  TEOS-10 releases.
- Do not modify `test/gsw_data_v3_0.nc`. The README explicitly says:
  "The data set gsw_data_v3_0.nc must not be tampered with." It is the
  validation oracle.
- Mixing distinct GSW versions across the bundle (e.g. one library
  against v3.04 and another against v3.06) can produce inconsistent
  ocean states; downstream packages don't typically pin a version, so
  let the bundle's checkout drive everything.
- The library does not declare a NetCDF dependency in
  `CMakeLists.txt`, but the test program reads NetCDF — running tests
  requires a Fortran NetCDF available at runtime.
- Files use a mix of `.f90` and `.F90` — `.F90` ones go through the
  preprocessor (e.g. `gsw_mod_check_data.F90`).

## Further reading

- TEOS-10 home: https://www.teos-10.org (algorithmic reference).
- In-repo: `README` (upstream description), `README.JEDI.md` (build).
- Related briefs: `jedi-knowledge/vader.md`, `jedi-knowledge/soca.md`,
  `jedi-knowledge/jedi-bundle.md`.
