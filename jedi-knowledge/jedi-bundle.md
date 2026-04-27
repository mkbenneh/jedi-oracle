# jedi-bundle

**Repository:** https://github.com/jcsda-internal/jedi-bundle
**Branch tracked:** develop
**Role in JEDI:** the umbrella build that pulls every JEDI repo together and orchestrates the cmake/ecbuild build of the full stack.

## What it is

The "bundle" is not a code library — it's a thin wrapper around `ecbuild`'s
bundle mechanism. Its `CMakeLists.txt` declares each JEDI repository (URL,
branch/tag, optional flags) and `ecbuild_bundle_initialize` /
`ecbuild_bundle_finalize` orchestrate cloning and building them in
dependency order. When you "build JEDI", you actually build the bundle.

## How it fits into the bundle

Every other repo in `jedi-knowledge/` is a sub-repository of the bundle.
Conceptually:

- **mandatory:** gsw, oops, jedi-model-data, vader, saber, crtm, ioda-data,
  ioda, ufo-data, ufo, fv3-jedi-lm, fv3-jedi-data, fv3-jedi, soca,
  coupling
- **conditional defaults-on:** mpas, mpas-jedi-data, mpas-jedi (via
  `BUILD_MPAS=ON`)
- **conditional defaults-off:** gsibec, rttov, oasim, ropp-ufo, iodaconv,
  pyiri-jedi (each has its own `BUILD_*` flag)

## Key files

- `CMakeLists.txt` — the manifest. The oracle reads this to know which
  repos exist, where to clone them from, and which branch/tag to use.
- `cmake/` — helper modules referenced via `CMAKE_MODULE_PATH`.
- `.github/workflows/` — CI definitions, including the nightly test
  workflow.

## Common tasks

- **What repos does the bundle include?** — read `CMakeLists.txt` and
  list every `ecbuild_bundle( PROJECT … )` line. `/getCode` does this
  for you.
- **Enable an optional package** — flip the corresponding
  `option(BUILD_<NAME> "..." OFF)` line to `ON`. `/getCode` will do this
  if you ask it to.
- **Update branch/tag for a sub-repo** — edit the `BRANCH` or `TAG`
  argument on its `ecbuild_bundle(...)` line. Then re-clone or
  `git checkout` inside the sub-repo dir.
- **Build the whole thing** — see jedi-tips.md and
  https://jedi-docs.jcsda.org/ for the canonical build instructions
  (depends on environment / module setup at your site).

## Gotchas

- The `mpas` entry pins a specific commit, not a tag — see
  `jedi-knowledge/mpas.md` for why.
- Several optional packages (rttov, oasim) require licensed source you
  must have access to before cloning will succeed.

## Further reading

- https://jedi-docs.jcsda.org/ — official JEDI docs (mirrored in
  `jedi-docs/` if cloned locally)
- `jedi-knowledge/<repo>.md` for any sub-repo
