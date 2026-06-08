# skylab

**Repository:** https://github.com/jcsda-internal/skylab
**Branch:** develop
**Role in JEDI workflow:** the experiment catalog — YAML definitions of experiments, algorithms, observations, models, and evaluation that EWOK consumes.

## What it is

The Skylab repo is the curated set of experiment definitions and supporting
configuration that JCSDA's reference end-to-end JEDI experiments are built
from. It is not an executable: it is the YAML library that EWOK reads when
the user runs `create_experiment.py`. From the README: *"contains experiment
definitions and additional utilities to support JEDI-Skylab experiments."*

## How it fits into the workflow stack

- **EWOK** consumes Skylab's `experiments/` YAMLs and `algorithms/`, `obs/`,
  `models/`, `eval/` building blocks to generate ecFlow or Cylc workflows.
- **R2D2** is the data backend Skylab experiments rely on for backgrounds,
  observations, and analyses.
- **simobs** is invoked by Skylab's HofX-style experiments when stand-alone
  observation simulation is needed.

EWOK's `JEDI_WORKFLOW` env var is supposed to point at the directory that
contains these workflow repos as siblings — i.e. the `jedi-workflow/`
directory in this oracle layout.

## Key directories

- `experiments/` — top-level experiment YAMLs, one per scenario
  (e.g. `geos-3dvar-c90.yaml`, `coupled-gfs-mom6-hofx.yaml`,
  `era5-analysis.yaml`, `mpas-3denvar.yaml`, `ingest-observations.yaml`).
  This is what the user names when they call `create_experiment.py`.
- `algorithms/` — DA algorithm building blocks (variational variants,
  ensemble methods, hybrid B, BUMP saber tooling, HTLM, HofX,
  forecast). YAML snippets included into experiments.
- `obs/` — observation configuration. `obs/ingest/` holds per-instrument
  ingest YAMLs (BUFR, satellite radiance, radio occultation, etc.).
- `models/` — model-specific configuration (FV3-GFS, GEOS, MPAS, MOM6, …).
- `eval/` — evaluation / diagnostics configuration.

## Key entry points

- `experiments/<name>.yaml` — the file you pass to `create_experiment.py`.
  Includes `algorithms/`, `obs/`, `models/`, `eval/` snippets.
- `experiments/README.md` — overview of available experiments.
- `algorithms/var.yaml`, `algorithms/hofx*.yaml`,
  `algorithms/forecast.yaml` — common DA algorithms.
- `obs/ingest/*.yaml` — per-observation-type ingest definitions.

## Common tasks

- **Run an existing experiment** — `create_experiment.py
  $JEDI_WORKFLOW/skylab/experiments/<name>.yaml` (see
  `jedi-knowledge/ewok.md`).
- **Add a new experiment** — copy an existing `experiments/*.yaml`,
  modify fields (`workflow engine`, dates, model, observations), and PR it.
- **Pick observations to ingest** — edit
  `experiments/ingest-observations.yaml` or compose a new ingest
  experiment from `obs/ingest/*.yaml` snippets.
- **Switch from ecFlow to Cylc** — set `workflow engine: cylcworkflow`
  in your experiment YAML.

## Gotchas

- Skylab YAMLs reference paths relative to `$JEDI_WORKFLOW`, `$JEDI_SRC`,
  `$JEDI_BUILD`, `$EWOK_WORKDIR`, and `$EWOK_FLOWDIR`. All must be
  exported before `create_experiment.py` will succeed.
- Some observation ingest paths require external credentials (NASA
  Earthdata, JAXA G-Portal, NCAR RDA via Globus). See `ewok.md` and
  the EWOK README for setup.
- **`adpsfc_ncep_prepbufr` now assimilated (2026-05):**
  `obs/ufo/adpsfc_ncep_prepbufr.yaml` was updated with expanded
  variable/channel settings so surface BUFR observations can be
  assimilated (skylab#901). If your experiments skip this type, no
  action needed; if you were working around the old incomplete YAML,
  remove the workaround.
- **Surface model updated in *_ldm.yaml obs files (2026-06, skylab#914):**
  `obs/ufo/buoy_ldm.yaml`, `metar_ldm.yaml`, `scatwind.yaml`,
  `ship_ldm.yaml`, and `synop_ldm.yaml` were all updated with a new
  surface model specification. If your experiments include any of these
  LDM observation types and you override these YAMLs locally, rebase
  against the new surface model settings.

## Further reading

- https://jedi-docs.jcsda.org/ → Inside JEDI → Skylab
- `jedi-knowledge/ewok.md`, `jedi-knowledge/r2d2.md`,
  `jedi-knowledge/simobs.md`, `jedi-knowledge/workflow.md`
