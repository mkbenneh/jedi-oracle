# oasim

**Repository:** https://github.com/jcsda-internal/oasim
**Branch tracked by bundle:** develop
**Role in JEDI:** Ocean–Atmosphere Spectral Irradiance Model used by SOCA / ocean DA observation operators. Optional bundle component.

## What it is

OASIM (Ocean–Atmosphere Spectral Irradiance Model) computes spectral
irradiance fields used in ocean optical / biogeochemistry observation
operators. It's invoked from UFO/SOCA when assimilating ocean color or
other optically-driven observations.

## How it fits into the bundle

- Built only when `BUILD_OASIM=ON` (default `OFF`).
- Consumed by ocean-side observation operators in `ufo` and/or `soca`.
- See `jedi-knowledge/soca.md` for ocean DA context.

## Common tasks

- **Enable OASIM** — pass `-DBUILD_OASIM=ON` to cmake, or let
  `/getCode` do it.

## Gotchas

- Internal repo — requires JCSDA-internal GitHub access to clone.

## Further reading

- `jedi-knowledge/soca.md`, `jedi-knowledge/ufo.md`
