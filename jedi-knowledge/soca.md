# soca

**Repository:** https://github.com/jcsda-internal/soca
**Branch tracked by bundle:** develop (cloned RECURSIVE)
**Role in JEDI:** the ocean-side OOPS Traits implementation — wraps MOM6 (and Icepack for sea ice) so JEDI's generic algorithms can drive ocean / sea-ice data assimilation.

## What it is

SOCA (Sea-ice Ocean Coupled Assimilation) is the ocean-DA companion to
fv3-jedi/mpas-jedi. Like those, it implements the OOPS Traits surface
(`Geometry`, `State`, `Increment`, `Model`, `LinearModel`,
`VariableChange`, `LinearVariableChange`, `ErrorCovariance`,
`ObsLocalization`, `IO`, …) — but for MOM6 ocean and Icepack sea-ice
configurations. Version 1.8.0.

The bundle clones SOCA `RECURSIVE`: the MOM6 model and Icepack live as
git submodules under `external/`, so SOCA pulls them along with itself.

## How it fits into the bundle

- **Depends on:** `oops`, `ufo`, `saber`, `vader`, `gsw`, `crtm` (some
  marine FOs); MOM6 and Icepack via submodules.
- **Depended on by:** `coupling` (combines SOCA with fv3-jedi for
  atm-ocean coupled DA).
- **Build position:** late, after the framework + after gsw.

## Key directories

- `src/soca/` — the SOCA library, organized like fv3-jedi/mpas-jedi:
  `Traits.h` at `src/soca/Traits.h`, plus subdirs for each Traits
  component (Geometry, State, Increment, Model, ErrorCovariance,
  VariableChange, LinearVariableChange, IO, …).
- `external/` — git submodules: MOM6 and Icepack.
- `bundle/CMakeLists.txt` — SOCA's own standalone bundle (used when
  building SOCA outside the larger jedi-bundle for ocean-only work).
- `test/` — 100+ YAML tests in `test/testinput/`. Test data is a
  72×35×25 tri-polar grid fixture under `test/Data/`.
- `docs/` — design notes.

## Key entry points

- `src/soca/Traits.h` — the SOCA Traits struct.
- `src/soca/Geometry/Geometry.{h,cc}` — wraps the MOM6 grid for
  Atlas / OOPS.
- `src/soca/State/State.{h,cc}`, `Increment/Increment.{h,cc}` — core
  state representation.
- `src/soca/Model/` — MOM6 forecast model wrapper.
- `src/soca/VariableChange/` — ocean variable transforms (often
  delegating to VADER's marine recipes when GSW is available).
- `src/mains/` — application drivers; canonical executables like
  `soca_var.x`, `soca_hofx.x`, `soca_forecast.x`, `soca_ensvariance.x`.

## Common tasks

- **Run the SOCA reference tests** — clone with `RECURSIVE` so MOM6 +
  Icepack come along, build, then `ctest -R soca_`.
- **Run a 3DVar experiment** — typically driven by skylab + ewok;
  executable is `soca_var.x` invoked with a YAML.
- **Update the MOM6 / Icepack pin** — `git submodule update --remote
  external/<sub>` and review the impact on SOCA's wrappers in
  `src/soca/Model/`.

## Gotchas

- **Recursive clone is required.** A non-recursive clone leaves
  `external/` empty and the build fails with missing-symbol errors
  for MOM6 and Icepack.
- Marine variable changes depend on `gsw` being in the build —
  without GSW, vader's marine recipes are silently disabled, which
  can change which recipe SOCA picks for a given variable.
- The 72×35×25 test grid is a low-res tri-polar configuration —
  realistic ocean DA experiments use higher-res configs from
  user-supplied data, not the bundled fixture.

## Further reading

- jedi-docs.jcsda.org → JEDI Components → SOCA.
- Related briefs: `jedi-knowledge/oops.md`, `jedi-knowledge/ufo.md`,
  `jedi-knowledge/vader.md`, `jedi-knowledge/saber.md`,
  `jedi-knowledge/gsw.md`, `jedi-knowledge/coupling.md`,
  `jedi-knowledge/jedi-bundle.md`.
