# Repository-specific Copilot instructions

**Purpose**: This repository contains a single analysis/demo notebook that produces several artifacts (plots and CSVs) demonstrating a quantum scheduling utility model using Qiskit/Aer. The guidance below helps an AI coding agent be productive editing, running, and extending this repo.

- **Entry point**: `quantum_scheduler_demo_aer_fast.ipynb` — treat this notebook as the canonical implementation. It also contains a header suggesting a runnable script name `quantum_scheduler_demo_aer_fast.py` and therefore can be converted to a script for CLI runs.
- **Artifacts directory**: `./artifacts` is the single output directory. Files created by the notebook include `decision_boundary.png`, `decision_boundary_smoke.png`, `adaptation_cumulative.png`, `adaptation_modes.png`, `nvqlink_latency_demo.png`, and `decision_boundary.csv`.

**Big picture / architecture**
- **Single-notebook workflow**: All logic lives in the notebook cells. High-level stages: imports & installs → presets/config (`FAST`, `SEED`, grids) → noise/timing model → circuit generation → simulation (Aer) → utility evaluation → plotting → writing artifacts.
- **Key components** (see cells in the notebook):
  - **Presets & flags**: `FAST`, `SEED`, `SMOOTHING_REPS` — these control speed/quality trade-offs.
  - **Simulator config**: `SIM_SV = AerSimulator(method='statevector')` with `SIM_SV.set_options(seed_simulator=SEED)` for deterministic runs.
  - **Noise model & caching**: functions decorated with `@lru_cache` — `_noise_model_cached`, `_layered_cached`, `_ideal_sv_cached`. Use `_q()` quantization helper to create stable cache keys.
  - **Compatibility helper**: `_sv_from_aer(qc, noise_model)` contains logic to support different Aer versions (tries `qc.save_statevector()` or `SaveStatevector` wrapper). When editing simulation code, keep this compatibility logic.

**Project-specific conventions & patterns**
- **Outputs**: always write to `OUTDIR = "./artifacts"` (created with `os.makedirs(OUTDIR, exist_ok=True)`). New outputs should follow the existing naming conventions.
- **Performance knobs**: when adding or testing, set `FAST = True` for quicker runs (smaller grids and fewer qubits). For full-quality runs set `FAST = False`.
- **Timing constants**: device-level constants (e.g., `DT`, `T1`, `T2`, `U1`, `U2`, `CX`, `MEAS`) are centralized near the top of the notebook — change them there for consistent behavior.
- **Determinism**: circuits use seeded RNGs (`np.random.default_rng(SEED)`) and Aer simulator seeding for reproducible outputs — don't remove seeds when debugging or producing deterministic artifacts.

**Dependencies & environment (how to reproduce)**
- The notebook itself installs dependencies at the top via `%pip install numpy pandas matplotlib qiskit qiskit-aer`. For terminal use prefer:

  ```bash
  python -m pip install --upgrade pip
  python -m pip install numpy pandas matplotlib qiskit qiskit-aer
  ```

- If you need a quick smoke test, open the notebook and run only the first few cells, or set `FAST = True` and execute. Converting to a script is supported: `jupyter nbconvert --to script quantum_scheduler_demo_aer_fast.ipynb` then run the generated `.py` (edit top cell magic `%pip` lines first).

**Common pitfalls & how to debug them (from code)**
- Aer compatibility: some Aer versions lack `save_statevector()` — `_sv_from_aer` handles fallbacks. If you see "No statevector found", inspect Aer version and the `SaveStatevector` import block.
- Slow runs: measurement time `MEAS` and classical boundary `CLASSICAL_BOUNDARY` dominate timing; use `FAST=True` and smaller grids when iterating.
- Caching surprises: because of `@lru_cache`, when experiments change global constants you may need to restart the kernel to invalidate caches or change the seed/inputs used as cache keys.

**Editing guidance for agents**
- Small edits: prefer changing notebook cells in-place. Keep configuration constants at top rather than scattering magic numbers.
- Adding functionality: prefer helper functions (follow naming examples `_noise_model_cached`, `_layered_cached`, `_cut_fidelity_circuit`) and use `@lru_cache` for reusable/expensive computations.
- Tests/Runs: there are no unit tests. Validate changes by running the notebook (FAST mode) and confirming expected artifact(s) are produced in `./artifacts`.

If any section is unclear or you want more detail (example: a minimal script runner, an explicit `requirements.txt`, or a reproducible Docker/venv recipe), tell me what to add and I will update this file.
