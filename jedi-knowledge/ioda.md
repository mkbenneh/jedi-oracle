# ioda

**Repository:** https://github.com/jcsda-internal/ioda
**Branch tracked by bundle:** develop
**Role in JEDI:** Interface for Observational Data Access — JEDI's observation-data layer. Defines `ObsSpace` (the in-memory representation of an observation set), the on-disk IODA file format, the parallel I/O engines, and the distribution machinery that splits observations across MPI ranks.

## What it is

IODA is the standardized observation I/O and in-memory data layer that
sits between raw observation files and JEDI's computational code. UFO
operates on `ioda::ObsSpace` objects; backgrounds and feedback round-trip
through IODA files. The IODA format is HDF5-based with strong type and
metadata conventions.

The library provides:

- A C++ `ObsSpace` API (the canonical in-memory representation).
- Multiple **engines** (HDF5 file backend, in-memory backend, ObsStore
  for SQL-like queries, and others).
- **Distributions** (Halo, RoundRobin, InefficientDistribution, …) that
  partition obs across MPI ranks.
- **Reader / writer / ioPool** infrastructure for parallel I/O.
- A standalone `IodaTrait.h` so apps that need only obs-side
  infrastructure (without UFO operators) can use IODA directly.

## How it fits into the bundle

- **Depends on:** `oops`, eckit, fckit, HDF5 (parallel), NetCDF, MPI.
- **Depended on by:** `ufo`, `iodaconv`, every model wrapper that runs
  DA against observations.
- **Build position:** after oops, before ufo.

## Key directories

- `src/` — the IODA library.
  - Top-level: `ObsSpace.{h,cc}`, `ObsVector.{h,cc}`,
    `ObsDataVector.h`, `ObsIterator.{h,cc}`,
    `ObsSpaceParameters.h`, `ObsDataIoParameters.{h,cc}`,
    `IodaTrait.h`.
  - `core/` — core types (variables, attributes, frames, named
    objects).
  - `containers/` — internal data containers.
  - `distribution/` — `DistributionFactory`, `Halo`,
    `RoundRobinDistribution`, `InefficientDistribution`, …
  - `engines/` — backend storage engines:
    - `engines/Engines/` — top-level engine selection.
    - `engines/Engines/HH/` — HDF5 backend.
    - `engines/Engines/ObsStore/` — in-memory ObsStore backend.
  - `example/` — small standalone examples.
- `share/` — installed shared resources.
- `test/` — gtest + ctest harness.
- `tools/` — utility scripts (e.g. dump utilities for IODA files).
- `docs/` — design notes.

## Key entry points

- `src/IodaTrait.h` — the standalone Traits header for apps using IODA
  without UFO.
- `src/ObsSpace.{h,cc}` — the primary user-facing class. Walking
  through `ObsSpace.cc` shows how files are opened, distributions are
  applied, and variables are loaded.
- `src/ObsSpaceParameters.h` — the YAML schema for `obs space:` blocks.
- `src/distribution/DistributionFactory.h` — registration entry point
  for distributions.
- `src/engines/Engines/Factory.h` — engine selection.
- `src/example/` — minimum-viable usage examples.

## Common tasks

- **Run the IODA tests** — `ctest -L ioda`.
- **Inspect an IODA file** — `tools/` contains dumpers; alternatively
  `h5dump`/`ncdump` work on an IODA HDF5 file.
- **Choose a distribution** — set `obs space.distribution.name:` in
  YAML to `Halo`, `RoundRobin`, etc. See `src/distribution/` for the
  available options and their characteristics.
- **Write a new engine** — derive from the base in
  `src/engines/Engines/`, register with the engine factory.
- **Convert raw obs to IODA format** — that's `iodaconv`'s job; see
  `jedi-knowledge/iodaconv.md`.

## Gotchas

- IODA files are HDF5 with specific group/variable conventions —
  arbitrary HDF5 files won't work. Use `iodaconv` outputs or another
  IODA-aware tool.
- Variable naming conventions matter: simulated values, observed
  values, errors, QC flags, and metadata each live in conventional
  groups (`hofx`, `ObsValue`, `ObsError`, `EffectiveQC`, `MetaData`,
  …). Mismatched names produce silent zeros downstream.
- `Halo` distribution is the typical choice for high-resolution global
  models; `RoundRobin` is a fallback for testing. Picking the wrong
  one can dramatically change MPI load balance.

## Further reading

- jedi-docs.jcsda.org → JEDI Components → IODA.
- Related briefs: `jedi-knowledge/oops.md`, `jedi-knowledge/ufo.md`,
  `jedi-knowledge/iodaconv.md`, `jedi-knowledge/ioda-data.md`,
  `jedi-knowledge/jedi-bundle.md`.
