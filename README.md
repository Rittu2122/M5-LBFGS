# A Limited-Memory M5 Quasi-Newton Method

This repository contains the implementation and numerical experiments for the manuscript **“A Limited-Memory M5 Quasi-Newton Method”** by Milan De and Manoj Kumar.

The proposed method, **M5–LBFGS**, combines the three-stage M5 method for nonlinear equations with an L-BFGS approximation of the inverse Hessian. The same inverse-Hessian approximation is used in all three stages. A damping parameter scales the complete three-stage step, and backtracking accepts a trial step when it satisfies both a sufficient-descent condition and the Armijo condition.

We compare M5–LBFGS with **L-BFGS (Wolfe)**, using the strong Wolfe conditions and SciPy's Moré–Thuente line search, and **L-BFGS (Armijo)**, a control method using the same backtracking and curvature-pair retention test as M5–LBFGS.

The convergence analysis establishes finite termination of backtracking, global convergence, and R-linear convergence for uniformly convex objective functions under the assumptions stated in the manuscript. Since many CUTEst test problems are nonconvex, the numerical experiments demonstrate empirical performance beyond the setting covered by the theory; no theoretical claim is made for those nonconvex problems.

---

## Contents

| File / folder | Content |
|---|---|
| `M5LBFGS_CUTEst_paper_experiments.ipynb` | Self-contained Google Colab notebook: setup, implementation of all three methods, experiments, and generation of the tables and figures in Section 6 and Appendix A of the manuscript |
| `results/` | Generated result files, including CSV and LaTeX tables, PDF and PNG figures, and `environment.txt` |

The notebook implements the L-BFGS two-loop recursion, the damped three-stage M5 step, Armijo backtracking, and the curvature-pair retention test. The Wolfe line search is SciPy's Moré–Thuente implementation (`scipy.optimize.line_search_wolfe1`).

---

## Test problems

We use unconstrained, twice continuously differentiable CUTEst problems whose dimension can be selected by the user. Problems are accessed through [PyCUTEst](https://github.com/jfowkes/pycutest), and each uses its standard starting point. Four problems are excluded: FLETCBV2 and MOREBV, because their standard starting points already satisfy the stopping test owing to objective scaling by the square of the mesh size; and FLETCBV3 and INDEF, because their objectives are unbounded below.

The two test sets are analysed separately because most problems appear in both sets at different dimensions.

| Test set | Dimension | Construction | Number of problems | Results prefix |
|---|---:|---|---:|---|
| Test set I | `1000 ≤ n ≤ 2996` | Each problem at its smallest available dimension in this range | 98 | `main` |
| Test set II | `5000 ≤ n ≤ 10000` | Each problem at its largest available dimension in this range | 66 | `large` |

CUTEst is described in Gould, Orban and Toint, *Computational Optimization and Applications* 60 (2015), 545–557. PyCUTEst is described in Fowkes, Roberts and Bürmen, *Journal of Open Source Software* 7(78) (2022), 4377.

---

## Experimental settings

| Setting | Value |
|---|---|
| Default L-BFGS memory size | `m = 5` |
| Memory-size study | `m = 3, 10, 20` on Test set I, compared with the default `m = 5` |
| Initial inverse-Hessian approximation | `H₀ = γI`, with `γ = sᵀr / rᵀr` after a correction pair is available and `γ_init = 1 / ‖g₀‖` before the first pair |
| Stopping test | `‖gₖ‖ < 10⁻⁵ max(1, ‖xₖ‖)`, following Liu and Nocedal (1989), equation (2.8) |
| Iteration limit | 50,000 iterations |
| L-BFGS (Wolfe) | Strong Wolfe conditions, `c₁ = 10⁻⁴`, `c₂ = 0.9`, starting with a unit step |
| L-BFGS (Armijo) and M5–LBFGS backtracking | `c₁ = 10⁻⁴`, backtracking factor `β = 1/2`, at most 60 trial values |
| Additional sufficient-descent condition for M5–LBFGS | `−gᵀd ≥ c_d α gᵀHg`, with `c_d = 10⁻⁴` |
| Curvature-pair retention in the two backtracking methods | Store a pair only if `sᵀr ≥ 10⁻¹⁰ sᵀs`; L-BFGS (Wolfe) stores every pair |
| Gradient evaluations | Counts all gradient calls, including gradients at intermediate M5 points and rejected trial values |
| CPU time | Process time. Every initially converged problem–method run is timed three times in total (initial run plus two repeats), regardless of duration; the median of timings from runs that converge is reported. An initially nonconverged run is timed once. |

A run fails if the stopping test is not met within 50,000 iterations, the Wolfe line search fails, or all 60 backtracking trial values are rejected. CPU times for failed runs are not included in the reported CPU-time summaries.

---

## How to run

1. Open `M5LBFGS_CUTEst_paper_experiments.ipynb` in [Google Colab](https://colab.research.google.com/) and select a CPU runtime; no GPU is used.
2. Choose **Runtime → Run all**. The setup cell installs the required tools for CUTEst and PyCUTEst. The notebook asks to mount Google Drive so results can be retained across Colab sessions.
3. The current notebook configuration saves results in the Google Drive folder `M5LBFGS_CUTEst_results_repeat_all_converged`, writing results as problems finish. If a runtime disconnects, reconnect to the same Drive folder and rerun the notebook; existing saved results can be reused.
4. The final download cell packages the available result files into `M5LBFGS_CUTEst_results.zip` and downloads the archive.

The complete experiment, including both test sets and the memory-size study, may take several hours depending on the Colab runtime.

---

## Where the paper's tables and figures come from

The notebook saves tables in CSV and LaTeX formats and figures in PDF and PNG formats. Output filenames use `_main` for Test set I and `_large` for Test set II.

| In the manuscript | Output file(s) |
|---|---|
| Table 1 — Test set I: summary and comparison | `table2a_convergence_main`, `table2b_statistics_main`, `table2c_summary_main` |
| Table 2 — Test set I: nonconverged runs and failure reasons | `table_failures_main` |
| Table 3 — Test set II: summary and comparison | `table2a_convergence_large`, `table2b_statistics_large`, `table2c_summary_large` |
| Table 4 — Test set II: nonconverged runs and failure reasons | `table_failures_large` |
| Table 5 — effect of the memory size | `table_memory` |
| Tables A1–A2 — per-problem results | `table1_per_problem_main`, `table1_per_problem_large` |
| Supplementary per-problem final objective values and gradient norms | `table1b_final_values_main`, `table1b_final_values_large` |
| Figures 1–3 — iteration, gradient-evaluation and CPU-time profiles for Test set I | `profile_iterations_main`, `profile_gradients_main`, `profile_cpu_main` |
| Figures 4–6 — iteration, gradient-evaluation and CPU-time profiles for Test set II | `profile_iterations_large`, `profile_gradients_large`, `profile_cpu_large` |
| Figure 7 — median iterations versus memory size | `memory_iterations_main` |
| Figure 8 — performance ratios versus memory size | `memory_ratios_main` |
| L-BFGS (Wolfe) implementation check against SciPy's L-BFGS-B (Section 6.2) | `scipy_check_main` |

The raw result tables `results_main.csv` and `results_large.csv` contain the per-problem, per-method results for `m = 5`, including dimensions, convergence status, failure reason, iterations, gradient and function evaluations, CPU time, final objective value, and final gradient norm. The memory study also produces `results_main_m3.csv`, `results_main_m10.csv`, and `results_main_m20.csv`.

The notebook also contains diagnostic cells that can generate extra files on step acceptance and CPU-time breakdown. These diagnostics are not presented as tables or figures in the current manuscript.

---

## Computing environment

The reported experiments were run on Google Colab using one thread on an Intel Xeon CPU at 2.20 GHz, with 12.7 GB of memory under Ubuntu 24.04.

The software versions specified in the manuscript are:

- Python 3.13
- NumPy 2.1.3
- SciPy 1.16.3
- PyCUTEst 1.8.2
- CUTEst 2.7.1
- SIFDecode 3.1.1
- gfortran 13.3.0

The generated `environment.txt` records additional environment details when available. To reproduce the exact reported run, use the software versions and CUTEst/SIFDecode/SIF library revisions recorded with the results. Iteration counts, gradient evaluations, convergence status, and final values are deterministic for the same implementation, inputs, and software environment. CPU timings may vary with machine and system load.

---

## Summary of results

The headline results in the manuscript are summarized below. Geometric-mean ratios are computed over problems solved by all three methods; a value below 1 favours M5–LBFGS for iterations, while a value above 1 indicates greater cost for gradient evaluations or CPU time.

| Measure | Test set I (98 problems) | Test set II (66 problems) |
|---|---:|---:|
| Problems solved: L-BFGS (Wolfe) / L-BFGS (Armijo) / M5–LBFGS | 84 / 87 / **89** | 60 / 61 / **63** |
| Geometric mean of iteration ratio: M5–LBFGS / L-BFGS (Wolfe) | **0.56** | **0.62** |
| Geometric mean of gradient-evaluation ratio: M5–LBFGS / L-BFGS (Wolfe) | 2.16 | 2.40 |
| Geometric mean of CPU-time ratio: M5–LBFGS / L-BFGS (Wolfe) | 1.71 | 1.97 |

M5–LBFGS solves more problems than either L-BFGS implementation in both test sets. On the problems solved by all three methods, it uses substantially fewer iterations (geometric-mean iteration ratios of 0.56 and 0.62), but requires more gradient evaluations per iteration. Its lower iteration count therefore does not generally translate into lower CPU time: M5–LBFGS is faster on 7 of the 84 common-solved problems in Test set I and 3 of the 59 common-solved problems in Test set II. The memory-size study on Test set I gives similar iteration and gradient-evaluation ratios for `m = 3, 5, 10, 20`.

---

## References

- D. C. Liu and J. Nocedal, “On the limited memory BFGS method for large scale optimization,” *Mathematical Programming* 45 (1989), 503–528.
- N. I. M. Gould, D. Orban and Ph. L. Toint, “CUTEst: a Constrained and Unconstrained Testing Environment with safe threads for mathematical optimization,” *Computational Optimization and Applications* 60 (2015), 545–557.
- J. Fowkes, L. Roberts and Á. Bürmen, “PyCUTEst: an open source Python package of optimization test problems,” *Journal of Open Source Software* 7(78) (2022), 4377.
