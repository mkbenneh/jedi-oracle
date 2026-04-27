# jedi-model-data

**Repository:** https://github.com/jcsda-internal/jedi-model-data
**Branch tracked by bundle:** develop
**Role in JEDI:** test-data sidecar for the model-agnostic JEDI components — provides fixtures consumed by `vader` and `saber` tests.

## What it is

A small data repo. The `CMakeLists.txt` declares an
`ecbuild_install_project` so downstream CMake projects can locate the
data via `find_package(jedi-model-data)`. Content lives under
`testinput_tier_1/`, organized by consumer (vader, saber, ...).

This repo is uncommonly important because vader's and saber's test
suites are *gated* on its presence: when `find_package(jedi-model-data)`
fails, those test suites are skipped silently — builds pass without
having actually exercised the libraries.

## How it fits into the bundle

- **Consumed by:** `vader` (test data and reference outputs), `saber`
  (test data and reference outputs).
- **Built when:** unconditionally cloned by the bundle (no opt-out
  flag).
- **Build position:** before vader and saber tests.

## Key directories

- `testinput_tier_1/saber/` — saber-side test fixtures (B-matrix
  reference data, dirac responses, etc.).
- `testinput_tier_1/vader/` — vader-side test fixtures (input
  FieldSets and expected output FieldSets for variable-change tests).

## Key files

- `CMakeLists.txt` — single `ecbuild_install_project` declaration.
- `README.md` — points at the consumers.

## Common tasks

- **Update fixtures for a vader or saber test** — modify under the
  appropriate `testinput_tier_1/<consumer>/` subtree. Bump the
  project version. Coordinate with the consumer repo's CMake
  `find_package` version requirement.
- **Confirm tests are actually running** — at configure time,
  vader/saber print whether `jedi-model-data` was found. Watch for
  *not found* messages — they mean tests will be silently skipped.

## Gotchas

- Silent test skipping is the main pitfall. If you've made changes to
  vader or saber and "tests still pass," check that
  `jedi-model-data` was found. A missing data repo means the test
  was never run.

## Further reading

- Related briefs: `jedi-knowledge/vader.md`, `jedi-knowledge/saber.md`,
  `jedi-knowledge/jedi-bundle.md`.
