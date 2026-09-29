# Claim scope — fixture-scoped QUALIFIED (H1-only)

## What is claimed

Under contract `regenie-v4.1.3-serial-eigen-step1-qt-n1000-m2293-v1` (fixture_revision 1),
the **H1-only** candidate (`experiments/H1_llt_per_lambda/`) achieves **≥2×** wall-clock
speedup vs the frozen Eigen-only serial baseline on the **named development fixture**:

- Input: `benchmarks/dev_fixture_step1_qt/inputs/regenie_dev_n1000` (N=1000, M=2293 PLINK bed)
- Envelope: `--threads 1` + `OMP/MKL/OPENBLAS/GOTO_NUM_THREADS=1` + `taskset -c 0`
- Build: Eigen-only Makefile (`HAS_BOOST_IOSTREAM=1`); **no MKL, no OpenBLAS, no `-march=native`**
- Timing boundary: process-start → process-exit
- Statistic: N=30 randomized paired rounds (seed `20260929`); median speedup (from medians)
  and one-sided 95% Student-t lower bound on mean(log speedup) both ≥ 2.0
- Correctness: `fit_1.loco` byte-identical; `fit_pred.list` basename-normalized

**Quoted from `opti2x/stats.json`:**

| Metric | Value |
|--------|------:|
| baseline median wall | 1.975 s |
| H1 median wall | 0.630 s |
| **Median speedup (medians)** | **3.135×** |
| Median paired speedup | 3.102× |
| Min paired speedup | 2.732× |
| Child faster | 30/30 |
| **One-sided 95% Student-t lower bound** | **3.054×** |

## What is **not** claimed

- **Not** a package-wide 2× claim for regenie.
- **Not** credit for `-march=native` / H3 (host-ISA; excluded from primary portable stack).
- **Not** a claim under multi-thread / uncapped OpenMP envelopes.
- **Not** MKL/OpenBLAS substitution as a win.
- **Not** LOOCV, binary-trait, or Cox step1 paths (fixture is QT 5-fold).
- **Not** H1+H3 ~8.5× stack as the QUALIFIED primary artifact.

## Credited optimization

**H1 only:** replace per-fold `SelfAdjointEigenSolver` + diagonal \((D+\lambda)^{-1}\) with
per-λ `LLT` solve of \((A+\lambda I)\beta = b\) inside `ridge_level_0`
(`src/Step1_Models.cpp`). Fairness patch: none.
