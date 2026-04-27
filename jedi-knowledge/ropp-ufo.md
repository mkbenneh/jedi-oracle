# ropp-ufo

**Repository:** https://github.com/jcsda-internal/ropp-test
**Branch tracked by bundle:** develop
**Role in JEDI:** ROPP (Radio Occultation Processing Package) integration for GNSS-RO observation operators in UFO. Optional bundle component.

## What it is

ROPP (Radio Occultation Processing Package) is the EUMETSAT GRAS SAF
toolkit for processing GPS/GNSS radio occultation observations
(bending angle, refractivity profiles). The `ropp-ufo` integration
lets UFO use ROPP's forward operators when assimilating GNSS-RO data.

## How it fits into the bundle

- Built only when `BUILD_ROPP=ON` (default `OFF`).
- Consumed by `ufo` as the GNSS-RO observation operator backend.
- See `jedi-knowledge/ufo.md` for the operator framework.

## Common tasks

- **Enable ROPP** — pass `-DBUILD_ROPP=ON` to cmake, or let `/getCode`
  do it.

## Gotchas

- Licensed software (like RTTOV) — full ROPP requires ROM SAF
  registration. The `ropp-test` repo is the JCSDA-internal wrapper.

## Further reading

- https://www.romsaf.org/ — upstream ROPP docs
- `jedi-knowledge/ufo.md`
