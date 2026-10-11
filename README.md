# A Limited-Memory M5 Quasi-Newton Method with Armijo Damping Globalization

This repository contains the implementation and the complete numerical experiments of the paper
*"A Limited-Memory M5 Quasi-Newton Method with Armijo Damping Globalization"*
(Milan De, Manoj Kumar; submitted to *Computational Optimization and Applications*).

The proposed method, **M5–LBFGS**, combines the three-stage M5 method for nonlinear equations with the
L-BFGS approximation of the inverse Hessian. One damping parameter scales the complete three-stage step,
and a step is accepted when it satisfies a sufficient-descent condition and the Armijo condition.
It is compared with **L-BFGS (Wolfe)**, the original method of Liu and Nocedal (1989), and
**L-BFGS (Armijo)**, a control that uses the same backtracking as M5–LBFGS.

---

## Contents

| File / folder | Content |
|---|---|
| `M5LBFGS_CUTEst_paper_experiments.ipynb` | One self-contained notebook: installation of CUTEst, the three methods, all experiments, and every table and figure of Section 6 and Appendix A of the paper |
| `results/` | All results produced by the notebook for the paper (`.csv`, `.tex`, `.pdf`, `.png`, `environment.txt`) |

The notebook contains its own implementation of the L-BFGS two-loop recursion, the damped M5 step,
the Armijo backtracking and the curvature test. The Wolfe line search is the Moré–Thuente line search
of SciPy (`scipy.optimize.line_search_wolfe1`). No other optimization package is used.

---

## Test problems

All unconstrained, twice continuously differentiable problems of variable dimension from the
[CUTEst](https://github.com/ralna/CUTEst) collection, accessed through
[PyCUTEst](https://github.com/jfowkes/pycutest), with their standard starting points.
Two test sets are formed and analysed separately:

| Test set | Dimension | Problems | Name in the files |
|---|---|---|---|
| Test set I | 1000 ≤ n ≤ 2996 (smallest available n ≥ 1000) | 98 | `main` |
| Test set II | 5000 ≤ n ≤ 10000 (largest available n in this range) | 66 | `large` |

Excluded problems: FLETCBV2 and MOREBV (the starting point already satisfies the stopping test,
because the objective is scaled by the square of the mesh size), and FLETCBV3 and INDEF
(objective unbounded below).

---

## Settings (identical for all methods)

| Setting | Value |
|---|---|
| Memory size | m = 5 (memory study: m = 3, 10, 20 on Test set I) |
| Initial matrix | H⁽⁰⁾ = γ I with γ = sᵀy / yᵀy (Liu–Nocedal strategy M3); γ = 1/‖g₀‖ before the first pair |
| Stopping test | ‖gₖ‖ < 10⁻⁵ · max(1, ‖xₖ‖)  (Liu and Nocedal 1989, eq. (2.8)) |
| Iteration limit | 50000 |
| Wolfe line search (L-BFGS (Wolfe)) | c₁ = 10⁻⁴, c₂ = 0.9 |
| Backtracking (L-BFGS (Armijo), M5–LBFGS) | c₁ = 10⁻⁴, β = 1/2, at most 60 trial values |
| Sufficient descent (M5–LBFGS) | −gᵀd ≥ c_d α gᵀHg with c_d = 10⁻⁴ |
| Curvature test (backtracking methods) | a pair is stored only if sᵀy ≥ 10⁻¹⁰ sᵀs |
| Gradient evaluations | every call of ∇f is counted, including the intermediate M5 points and rejected trial values |
| CPU time | process time; runs shorter than 5 s are repeated three times and the median is reported |

A run fails if the stopping test is not met within 50000 iterations, if the Wolfe line search fails,
or if all 60 trial values of the backtracking are rejected.

---

## How to run

1. Open `M5LBFGS_CUTEst_paper_experiments.ipynb` in [Google Colab](https://colab.research.google.com/)
   (CPU runtime; no GPU is used).
2. Choose **Runtime → Run all**.
   - Part 1 installs gfortran, SIFDecode, CUTEst, the SIF problem library and PyCUTEst (about 5–10 minutes).
   - Part 2 asks for access to Google Drive. The results are saved in the Drive folder
     `M5LBFGS_CUTEst_results` after every problem.
3. If Colab disconnects, choose **Run all** again: problems that are already saved are skipped, and the
   computation continues where it stopped. When all results are saved, **Run all** only rebuilds the
   tables and figures (a few minutes).
4. The last cell writes all results into `M5LBFGS_CUTEst_results.zip` and downloads it.

The complete experiment (both test sets, the memory study and the additional experiments) takes several hours.

---

## Where each table and figure of the paper comes from

| Paper | File in `results/` |
|---|---|
| Table 1 (Test set I: summary and comparison) | `table2a_convergence_main`, `table2b_statistics_main`, `table2c_summary_main` |
| Table 2 (Test set I: runs that did not converge) | `table_failures_main` |
| Table 3 (Test set II: summary and comparison) | `table2a_convergence_large`, `table2b_statistics_large`, `table2c_summary_large` |
| Table 4 (Test set II: runs that did not converge) | `table_failures_large` |
| Table 5 (effect of the memory size) | `table_memory` |
| Table 6 (damping and cost per iteration) | `table_damping`, `table_time_split` |
| Tables A1, A2 (results for every problem) | `table1_per_problem_main`, `table1_per_problem_large` |
| Figs. 1–3 (Test set I: performance profiles) | `profile_iterations_main`, `profile_gradients_main`, `profile_cpu_main` |
| Figs. 4–6 (Test set II: performance profiles) | `profile_iterations_large`, `profile_gradients_large`, `profile_cpu_large` |
| Fig. 7 (median iterations vs memory size) | `memory_iterations_main` |
| Fig. 8 (ratios M5–LBFGS / L-BFGS (Wolfe) vs memory size) | `memory_ratios_main` |
| Fig. 9 (accepted step lengths) | `damping_alpha_main` |
| Check of L-BFGS (Wolfe) against SciPy's L-BFGS-B (Section 6.2) | `scipy_check_main` |

Raw results of every run: `results_main.csv`, `results_large.csv` (m = 5) and
`results_main_m3.csv`, `results_main_m10.csv`, `results_main_m20.csv` (memory study).
For every problem and method they contain the dimension, convergence status, reason for stopping,
iterations, gradient and function evaluations, CPU time, final function value and final gradient norm.
`table1b_final_values_*` lists the final function values and gradient norms side by side.

---

## Computing environment of the reported results

The results in `results/` and in the paper were produced on Google Colab
(the same information is printed by the notebook and saved in `results/environment.txt`):

| | |
|---|---|
| Machine | Intel Xeon CPU @ 2.20 GHz, 2 logical cores (1 thread used), 12.7 GB RAM |
| OS | Linux 6.6.122+ (Ubuntu 24.04.5 LTS) |
| Software | Python 3.13.16, NumPy 2.1.3, SciPy 1.16.3, pandas 2.2.3, Matplotlib 3.10.0, PyCUTEst 1.8.2 |
| CUTEst | CUTEst 2.7.1 (commit 7dafe08), SIFDecode 3.1.1 (commit 979967c), SIF library (commit 319d16b), gfortran 13.3.0 |

**Reproducibility.** The three methods are deterministic, and the notebook contains no randomness.
Iteration counts, gradient and function evaluations, convergence status and final values are therefore
reproducible with the same software versions. CPU times depend on the machine and its load; they are
not expected to match exactly, but the comparison between the methods, which ran on the same machine,
is stable. Part 1 installs the current versions of CUTEst, SIFDecode and the SIF library; to reproduce
the exact numbers, check out the commits listed above.

---

## Summary of results

| | Test set I (98 problems) | Test set II (66 problems) |
|---|---|---|
| Problems solved: L-BFGS (Wolfe) / L-BFGS (Armijo) / M5–LBFGS | 84 / 87 / **89** | 60 / 61 / **63** |
| Iterations, M5–LBFGS / L-BFGS (Wolfe), geometric mean | **0.56** | **0.62** |
| Gradient evaluations, M5–LBFGS / L-BFGS (Wolfe) | 2.16 | 2.40 |
| CPU time, M5–LBFGS / L-BFGS (Wolfe) | 1.67 | 1.96 |

M5–LBFGS solves more problems than L-BFGS, never fails on a problem that L-BFGS solves, and needs
about 40% fewer iterations, independently of the memory size. Each iteration needs more gradient
evaluations, so L-BFGS is faster in CPU time. The convergence theory of the paper (global and R-linear
convergence) holds for uniformly convex functions; many CUTEst problems are nonconvex, so the
experiments also show the behaviour of the method beyond the setting of the theory.

---

## References

- D.C. Liu, J. Nocedal, On the limited memory BFGS method for large scale optimization,
  *Math. Program.* 45 (1989) 503–528.
- N.I.M. Gould, D. Orban, Ph.L. Toint, CUTEst: a Constrained and Unconstrained Testing Environment with
  safe threads for mathematical optimization, *Comput. Optim. Appl.* 60 (2015) 545–557.
- J. Fowkes, L. Roberts, Á. Bűrmen, PyCUTEst: an open source Python package of optimization test problems,
  *J. Open Source Softw.* 7(78) (2022) 4377.
