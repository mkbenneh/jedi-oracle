# rttov

**Repository:** https://github.com/jcsda-internal/rttov (mirror of upstream)
**Branch tracked by bundle:** develop
**Role in JEDI:** Radiative Transfer for TOVS — one of the radiative transfer backends UFO uses for satellite observation operators. Optional bundle component.

## What it is

RTTOV (Radiative Transfer for TOVS) is a fast radiative transfer model
maintained by the [NWP SAF](https://nwp-saf.eumetsat.int/site/software/rttov/).
It computes top-of-atmosphere radiances and brightness temperatures for
many satellite instruments and is widely used in operational NWP. The
JCSDA mirror in `jcsda-internal/rttov` exists because RTTOV is licensed
software — access requires registration with EUMETSAT.

In JEDI, RTTOV is one of two radiative transfer engines available to
UFO observation operators (the other is CRTM). Different agencies prefer
different RT models; both are optional and selected at runtime in the
observation operator config.

## How it fits into the bundle

- Built only when `BUILD_RTTOV=ON` (default `OFF`).
- Consumed by `ufo` as a radiative transfer backend for satellite
  radiance operators.
- See `jedi-knowledge/ufo.md` for the operator side and
  `jedi-knowledge/crtm.md` for the alternative RT engine.

## Common tasks

- **Enable RTTOV** — pass `-DBUILD_RTTOV=ON` to cmake, or let
  `/getCode` do it.
- **Get RTTOV access** — register with the NWP SAF (link above) for
  the source/coefficient files. The `jcsda-internal/rttov` mirror is
  internal-only.

## Gotchas

- **Licensed source.** Cloning the JCSDA mirror requires JCSDA-internal
  GitHub access; running RTTOV requires the official coefficient files
  from NWP SAF.

## Further reading

- https://nwp-saf.eumetsat.int/site/software/rttov/ — upstream docs
- `jedi-knowledge/ufo.md` and `jedi-knowledge/crtm.md`
