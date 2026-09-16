# A Limited-Memory M5 Quasi-Newton Method with Armijo Damping Globalization

This repository contains the reference implementation and reproduction notebooks for the paper *"A Limited-Memory M5 Quasi-Newton Method with Armijo Damping Globalization."* It provides a from-scratch implementation of **BFGS**, **L-BFGS**, and the proposed **M5-LBFGS** method, and reproduces every numerical table and figure reported in the paper.

## Contents

| File | Reproduces | Setting |
|---|---|---|
| `Andrei_Suite_M5LBFGS.ipynb` | Table 1 (per-problem results), Table 2 (aggregate summary), Table 3 (termination reasons), Figure 1 (Dolan–Moré performance profiles) | 30-problem Andrei unconstrained test collection, $n=500$, memory $m=20$ |
| `ill_conditioned_quadratic_comparison.ipynb` | Table 4 (spectral-conditioning sensitivity study) | SPD quadratic $f(x)=\tfrac12x^TAx$, $n=100$, $m=20$, $\kappa(A)\in\{10,10^2,10^3,10^4\}$ |

Both notebooks are **fully self-contained**: each includes its own implementation of the L-BFGS two-loop recursion, the strong-Wolfe line search, and the damped M5–LBFGS construction, so neither depends on the other or on any package outside the list below. Only the three methods reported in the paper — **BFGS**, **L-BFGS**, and **M5-LBFGS** — are included; no exploratory or unpublished variants are present.

## Requirements

- Python ≥ 3.10
- [PyTorch](https://pytorch.org/) (CPU build is sufficient — no GPU is used anywhere in either notebook)
- NumPy
- pandas
- Matplotlib

Every gradient used in these experiments is computed via PyTorch's automatic differentiation rather than a hand-coded analytic gradient, which removes gradient-implementation error as a possible source of discrepancy.

## Running the notebooks

Both notebooks run top-to-bottom with no configuration required, in any standard Jupyter environment:

- **Google Colab** — upload the notebook and select a CPU runtime (Runtime → Change runtime type → CPU). No GPU is needed or used.
- **Jupyter / JupyterLab** — `pip install torch numpy pandas matplotlib` then open normally.
- **VS Code** — open with the Jupyter extension installed and run all cells.

The first code cell of each notebook prints an environment fingerprint (Python version, PyTorch version, platform, CPU model, thread count). This is not required for the code to run — it is printed purely so the exact software/hardware environment behind a given set of results is always on record.

**Reproducibility note:** the exact numbers reported in the accompanying paper were produced in VS Code on Windows 10 (Python 3.14.2, PyTorch 2.13.0, Intel Core i3-6006U, 4 GB RAM). We verified that outer-iteration counts, gradient-evaluation counts, and convergence status reproduce *exactly* on Google Colab (CPU runtime) as well; CPU wall-clock time is expected to vary by environment and should not be compared directly across machines.


**CPU time is the one exception.** Wall-clock time reflects operating-system scheduling and instantaneous machine load in addition to the algorithm itself, so it is not expected to match exactly even on identical hardware. To reduce (not eliminate) this noise, CPU time for every run that converges is averaged over 10 repetitions; a run that exhausts its iteration budget without converging is timed once, since its cost is already fixed by a deterministic iteration count rather than by run-to-run variation.

## Summary of results

At $n=500$, $m=20$ (30-problem Andrei benchmark, Table 2):

| Method | Success rate | Mean iterations | Mean gradient evaluations |
|---|---|---|---|
| BFGS | 20/30 (66.7%) | 819.1 | 894.6 |
| L-BFGS | 23/30 (76.7%) | 332.2 | 443.1 |
| M5-LBFGS | 29/30 (96.7%) | 140.5 | 776.6 |

M5-LBFGS converges on the largest number of problems and uses substantially fewer outer iterations on the common successful set, at the cost of additional gradient evaluations per accepted step (three per trial versus one for BFGS/L-BFGS). See the paper for the complete discussion, including the spectral-conditioning sensitivity study (Table 4), where this iteration-level advantage narrows as the condition number of the problem grows.



