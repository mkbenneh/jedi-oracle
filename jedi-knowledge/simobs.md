# simobs

**Repository:** https://github.com/jcsda-internal/simobs
**Branch:** develop
**Role in JEDI workflow:** stand-alone observation simulation — runs UFO/CRTM forward operators on synthetic backgrounds without needing the full JEDI executable stack.

## What it is

SIMOBS is a high-performance stand-alone application for running JCSDA's
UFO and CRTM models against backgrounds. From the README: *"a
high-performance stand-alone operation and visualization application for
the JCSDA UFO and CRTM models."* It exists so users (and CI) can simulate
synthetic observations and exercise observation operators without
spinning up a full DA cycle.

## How it fits into the workflow stack

- **Wraps `ufo` and `crtm`** from the bundle. SIMOBS doesn't reimplement
  forward operators — it links against the same UFO/CRTM you build via
  jedi-bundle.
- **EWOK** can call SIMOBS as a workflow task in observation-ingest or
  HofX-style suites.
- **R2D2** is the typical data backend; SIMOBS reads backgrounds and
  writes feedback files into R2D2.

## Key directories

- `src/simobs/` — the SIMOBS Python package (and any C++/Fortran
  extensions).
- `data/` — supporting data and configuration.
- `notebooks/` — Jupyter notebooks demonstrating use and visualization.
- `test/` — unit tests.

## Key entry points

- `setup.py` — installs the `simobs` Python package and any console
  scripts.
- `src/simobs/` — start here to see the public API.
- `notebooks/` — best place to learn the user-facing flow by example.

## Common tasks

- **Run SIMOBS against a background** — typically via an EWOK task or
  directly from the installed CLI; see `notebooks/` for current
  invocation patterns since the README is short.
- **Inspect SIMOBS output** — feedback files land in R2D2 (when EWOK
  drives it) or in a user-specified directory for ad-hoc runs.

## Gotchas

- The README is sparse — the canonical reference is the Python source
  and the notebooks.
- The internal Slack channel referenced in the README
  (`jcsda.slack.com/archives/C02TR52N0B0`) is the active place for
  current usage questions.

## Further reading

- README + `notebooks/` in this repo.
- `jedi-knowledge/ufo.md`, `jedi-knowledge/crtm.md` for the underlying
  forward-operator code.
- `jedi-knowledge/ewok.md`, `jedi-knowledge/skylab.md`,
  `jedi-knowledge/workflow.md`.
