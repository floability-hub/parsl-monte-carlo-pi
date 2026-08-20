# Third-Party Notice

The Monte Carlo pi calculation and its fan-out/fan-in workflow are adapted
from the **Monte Carlo workflow** section of `1-parsl-introduction.ipynb` in
the Parsl tutorial:

- Exact source:
  https://github.com/Parsl/parsl-tutorial/blob/71fbd34d826bf60174fbab3a5213e4e9ed80640f/1-parsl-introduction.ipynb
- Source revision: `71fbd34d826bf60174fbab3a5213e4e9ed80640f`
- Upstream license: Apache License 2.0

The TaskVine configuration is based on Parsl's official local executor example:

- Exact source:
  https://github.com/Parsl/parsl/blob/b8b474c7a6ee6fd2f9392e10291c2d8c323b0510/parsl/configs/vineex_local.py

Floability-specific modifications include TaskVine executor configuration,
manual worker mode, environment-based manager discovery, permitted-port
selection, deterministic seeds, result validation, JSON output, and execution
metadata. The configurable larger ensemble is a Floability-specific extension
and is presented after the retained three-task tutorial example.
