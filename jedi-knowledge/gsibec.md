# gsibec

**Repository:** https://github.com/geos-esm/GSIbec
**Tag tracked by bundle:** 1.4.2
**Role in JEDI:** GSI-derived background error covariance package for use with SABER. Optional bundle component.

## What it is

GSIbec packages the background-error covariance routines from the legacy
GSI (Gridpoint Statistical Interpolation) system as a standalone library
that SABER can call. It lets JEDI users apply NCEP/GSI-style B
preconditioning without depending on the full GSI codebase.

Maintained by NASA GMAO (geos-esm), tagged for stability. The bundle
pins to a specific tag (`1.4.2`) rather than tracking `develop`.

## How it fits into the bundle

- Built only when `BUILD_GSIBEC=ON` (default `OFF`).
- Consumed by `saber` as one of several B-matrix backends.
- See `jedi-knowledge/saber.md` for how SABER selects between
  covariance backends.

## Common tasks

- **Enable the GSIbec backend** — pass `-DBUILD_GSIBEC=ON` to cmake, or
  let `/getCode` do it for you.
- **Bump the tag** — edit `TAG 1.4.2` in `jedi-bundle/CMakeLists.txt`.
  Coordinate with the SABER team since the SABER↔GSIbec interface
  changes over releases.

## Gotchas

- Pinned to a tag, not `develop` — `/updateRepos` won't pull it. To
  update, change the tag in the bundle CMakeLists.

## Further reading

- https://github.com/geos-esm/GSIbec
- `jedi-knowledge/saber.md` for how it's used
