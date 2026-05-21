# ewok

**Repository:** https://github.com/jcsda-internal/ewok
**Branch:** develop
**Role in JEDI workflow:** the orchestrator — turns Skylab experiment YAMLs into runnable ecFlow or Cylc workflows.

## What it is

EWOK = **Experiments and Workflows Orchestration Kit**. From the README:
EWOK reads experiment YAMLs (typically from Skylab), generates a workflow
suite, and submits it to a workflow engine. The engine is currently
[ecFlow](https://confluence.ecmwf.int/display/ECFLOW), with
[Cylc](https://cylc.github.io/) support under development.

## How it fits into the workflow stack

- **Reads from `skylab/`** — experiment YAMLs and the `algorithms/`,
  `obs/`, `models/`, `eval/` snippets they include.
- **Reads/writes via `r2d2/`** — backgrounds, observations, analyses,
  and diagnostics flow through R2D2's MySQL-backed archive.
- **Calls JEDI executables** in `$JEDI_BUILD` (built from `jedi-bundle/`).
- **Optionally drives `simobs/`** for stand-alone obs simulation tasks.

EWOK is the user-facing entry point; once an experiment is created, the
workflow engine takes over.

## Key directories

- `src/ewok/` — main EWOK Python package (suite generation, task
  scripting, R2D2 hooks).
- `src/runtime/` — runtime helpers loaded inside ecFlow/Cylc tasks.
- `src/yamltools/` — YAML composition utilities (substitution,
  inclusion, validation).
- `experiments/` — example experiments deployable on Derecho, Discover,
  Orion/Hercules, S4, and AWS.
- `test/` — pytest suite.
  - `test_cylc_flow_generation.py` — Cylc workflow generation tests.
  - `test_ecflow_def_generation.py` — ecFlow `.def` generation tests
    (added 2026-04; see `test/testdata/*.def` for reference suites).
  - `test/testdata/` — reference `.def` files: `gfs-3dvar-c24.def`,
    `gfs-hofx-c24.def`, `jedi-ctest.def`, `skylab-atm-land-small.def`,
    `workflow-engine-test.def`.
  - `conftest.py` — shared pytest fixtures (significantly expanded in
    2026-04 to support both Cylc and ecFlow tests).

## Key entry points

- `create_experiment.py` (installed by `setup.py`) — primary CLI:
  `create_experiment.py <experiment_file>.yaml`. Flags include
  `-ns/--no-submit`, `-sus/--suspend`, `--test`.
- `src/ewok/` — the Python module the CLI delegates to.
- `environment.yml` — conda environment definition for users not on a
  spack-stack platform.

## Required environment

Four env vars must be set before running any EWOK command (excerpt from
the README):

- `JEDI_WORKFLOW` — directory containing the workflow source repos
  (`ewok`, `r2d2-client`, `simobs`, `skylab`).
- `JEDI_SRC` — directory containing `jedi-bundle/`'s sub-repos.
- `JEDI_BUILD` — built JEDI executables.
- `EWOK_WORKDIR` — runtime/data dir for an experiment (large, often
  cluster-mounted scratch).
- `EWOK_FLOWDIR` — ecFlow suite definitions and task scripts.

For Cylc, also configure `~/.cylc/flow/global.cylc` per the JEDI
Knowledge Base notes referenced in the README.

## Common tasks

- **Create + submit an experiment** — `create_experiment.py
  $JEDI_WORKFLOW/skylab/experiments/<name>.yaml`.
- **Generate the suite without submitting** — add `-ns`.
- **Suspend automatic completion** (training/tutorials) — add `-sus`.
- **Generate CI test suite** — add `--test`.
- **Start the ecFlow server** — `ecflow_start.sh`; UI: `ecflow_ui &`.

## Gotchas

- The four env vars are not optional. Missing any one of them fails
  experiment creation in confusing ways.
- ecFlow server-side and client-side hostnames must match — when the UI
  doesn't see the server, add it manually under
  *Servers → Manage servers*.
- Cylc support requires extra Python dependencies; see the JEDI
  Knowledge Base wiki link in the README.
- Globus / NASA Earthdata / JAXA / RDA each have their own auth setup
  for ingest tasks. The README has detailed walkthroughs — start there
  rather than improvising.
- **Common prepbufr handling refactored (2026-05):**
  `src/ewok/tasks/convertObservations.py` and `storeObsBias.py` were
  updated to share common prepbufr handling logic (ewok#1282). If you
  have local branches touching these task files, check for merge
  conflicts.

## Further reading

- README in this repo (extensive — covers ingest, paths, Cylc setup,
  troubleshooting).
- https://jedi-docs.jcsda.org/ → Inside JEDI → Skylab.
- `jedi-knowledge/skylab.md`, `jedi-knowledge/r2d2.md`,
  `jedi-knowledge/simobs.md`, `jedi-knowledge/workflow.md`.
