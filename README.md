# Parsl Monte Carlo Pi

## Overview

This Floability backpack adapts the Monte Carlo workflow from the official
[Parsl tutorial](https://github.com/Parsl/parsl-tutorial). It first preserves
the tutorial's simple demonstration: three independent Parsl Python Apps
estimate pi by dropping random points into a unit square, and a fourth App
computes their mean. A second section then launches a configurable, larger
ensemble to show how additional independent samples improve accuracy.

The original tutorial calculation and fan-out/fan-in structure are retained.
This backpack adds deterministic random seeds, result validation, a JSON
summary, and a Parsl `TaskVineExecutor` configured to use the workers that
Floability launches.

## Upstream Source and Adaptation

The original code comes from the **Monte Carlo workflow** section of
[`1-parsl-introduction.ipynb`](https://github.com/Parsl/parsl-tutorial/blob/71fbd34d826bf60174fbab3a5213e4e9ed80640f/1-parsl-introduction.ipynb).
The executor setup follows Parsl's official
[`vineex_local.py`](https://github.com/Parsl/parsl/blob/b8b474c7a6ee6fd2f9392e10291c2d8c323b0510/parsl/configs/vineex_local.py)
example and the
[`TaskVineExecutor` documentation](https://parsl.readthedocs.io/en/stable/stubs/parsl.executors.taskvine.TaskVineExecutor.html).

For Floability adoption, we kept the three `pi()` Apps and dependent `mean()`
App as the first result, then made these small changes:

- added fixed random seeds for repeatable validation;
- configured `TaskVineExecutor` for manual workers so Floability remains the
  only `vine_factory` launcher;
- read the manager name and permitted ports from Floability environment
  variables;
- report each task's manager-observed TaskVine worker IP and connection port;
  and
- added a separate configurable ensemble, accuracy comparison, failure checks,
  timing, and a JSON result summary.

The scaled section defaults to 20 new tasks with 1,000,000 points each. Change
`SCALED_TASKS` in the notebook configuration cell to any value from 3 through
1,000. `MAX_OUTPUT` limits each stage's printed task-to-worker rows and defaults
to 20; the complete assignment list is still written to the JSON summary.
Increasing the number of tasks increases the total number of sampled points;
increasing worker count only changes how quickly those tasks finish.

## Install Floability

Install and activate Floability by following the
[official installation instructions](https://floability.readthedocs.io/en/stable/getting-started/installation/).
Verify the installation before running the backpack:

```bash
floability --version --verbose
```

## Run the Backpack

Run these commands from the backpack root.

### Local workers

```bash
floability run --backpack .
```

Open the JupyterLab URL printed by Floability, run the notebook cells in order,
save the notebook, and press `Ctrl+C` in the Floability terminal when finished.

### HTCondor workers

```bash
floability run --backpack . --batch-type condor
```

### Slurm workers

```bash
floability run --backpack . --batch-type slurm
```

The selected batch system must already be installed and configured at the
execution site.

### Non-interactive execution

Run every notebook cell without opening a browser:

```bash
floability execute --backpack . \
  --sync-path outputs/monte-carlo-summary.json
```

Add `--batch-type condor` or `--batch-type slurm` for an HPC scheduler.
The explicit sync path copies only the generated JSON summary back into the
backpack. Parsl and TaskVine diagnostic logs remain in the Floability instance.
Without the sync path, the summary also remains in the instance.

## Expected Result

A successful run first submits the official-style three independent `pi()`
Apps and one dependent `mean()` App. It prints those estimates and their
average, followed by each π estimate and the worker endpoint that produced it.
The next section submits 20 new `pi()` Apps by default, reduces them to a second
average, and compares the absolute errors and estimated sampling uncertainty.
It also prints up to `MAX_OUTPUT` scaled task-to-worker assignments before
writing:

```text
outputs/monte-carlo-summary.json
```

The deterministic seeds make the estimates repeatable for a fixed Python
version. The notebook fails if an App fails, a result is not finite, or an
average differs from `math.pi` by more than `0.02`.

The displayed `worker=IP:PORT` value is the connection endpoint observed by
the TaskVine manager. Parsl does not expose TaskVine's native `done.addrport`
property through an App future, so the notebook obtains the equivalent mapping
from TaskVine's transaction log.

Monte Carlo uncertainty decreases approximately with the square root of the
total number of sampled points. The default 20-task scaled stage samples
20,000,000 points, compared with 3,000,000 points in the introductory stage.

## How Parsl and Floability Work Together

Floability prepares the environment and starts `vine_factory`. The notebook
creates Parsl's TaskVine manager with the manager name supplied through
`VINE_MANAGER_NAME`.

The Parsl executor uses:

```python
TaskVineExecutor(worker_launch_method="manual", ...)
```

Manual worker mode is important: it prevents Parsl from starting a second
factory. Floability remains responsible for local, HTCondor, or Slurm worker
submission through `compute/compute.yml` and CLI site settings.

## Common Options

HPC home directories often have limited quotas. Place instances and prepared
environments on project or scratch storage, and constrain network ports to
ranges allowed by the site:

```bash
floability run --backpack . \
  --batch-type slurm \
  --base-dir "$SCRATCH/floability" \
  --manager-ports 9123:9150 \
  --worker-transfer-ports 10000:11000
```

- `--base-dir PATH` changes the root for instances, prepared environments,
  packed archives, logs, and the default data cache.
- `--manager-ports START:END` constrains the port used by Parsl's TaskVine
  manager. The notebook selects one currently available port in this range.
- `--worker-transfer-ports START:END` constrains worker-to-worker transfers.

Worker counts and core requests belong in `compute/compute.yml`.

## Backpack Contents

- `workflow/parsl-monte-carlo.ipynb` — Parsl/TaskVine configuration, the
  adapted Monte Carlo workflow, validation, and output writing.
- `software/environment.yml` — pinned Python, Parsl, TaskVine, and Jupyter
  environment.
- `compute/compute.yml` — four to eight local or scheduler-submitted workers
  with one core each.
- `LICENSE` — Apache License 2.0 from the upstream Parsl tutorial.
- `THIRD_PARTY_NOTICES.md` — source and modification notice for the adapted
  tutorial code.
