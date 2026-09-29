# Profile post-H1 — regenie v4.1.3 serial step1 QT fixture

**Binary:** H1-only `experiments/H1_llt_per_lambda/worktree/regenie`  
sha256 `4d3ecff43a46a079f6e869d22ff81f175db7c72c698a6461d0f074e29f3c2d8d`  
**Envelope:** `--threads 1`, `OMP/MKL/OPENBLAS/GOTO_NUM_THREADS=1`, `taskset -c 0`  
**Workload:** warm cache + one timed step1 on `dev_fixture_step1_qt`  
**Tool:** `perf record -F 999 -g` → `research/artifacts/perf_profile_post_H1/`  
**Date:** 2026-09-29 PT  
**Wall:** 0.63 s (log elapsed 0.635 s)

---

## Application-level timers (from `fit.log`)

| phase | ms (sum) | ≈% of 0.635 s |
|-------|---------:|-------------:|
| residualize/scale genotypes | 12 | 1.9% |
| calc working matrices (`calc_cv_matrices`) | **181** | **28.5%** |
| **calc level 0 ridge (`ridge_level_0`, now LLT/λ)** | **401** | **63.1%** |
| level 1 ridge | 0 | ~0% |
| write LOCO | 4 | 0.6% |

Per-block L0 ridge: 98, 92, 97, 94, 20 ms (last block smaller).  
Per-block working matrices: 47, 39, 44, 38, 13 ms.

**Contrast vs baseline profile** (`PROFILE.md`, elapsed ≈1.90 s): L0 ridge **1670 → 401 ms**; working matrices unchanged (~173–181 ms). Eig path eliminated as the dominant leaf.

---

## `perf report` — flat self (top)

Samples: 625 task-clock; lost samples 0.

| ~self % | samples | symbol (summary) | role |
|--------:|--------:|------------------|------|
| **~75.5%** | 472 | `gebp_kernel` (both constprop clones) | Eigen GEMM (`calc_cv_matrices` / related) |
| **5.76%** | 36 | `LLT::compute` | Cholesky of `(ww1+λI)` |
| **2.72%** | 17 | `readChunkFromBedFileToG` `._omp_fn.0` | bed decode |
| ~3.5% | — | `gemm_pack_lhs/rhs` | GEMM packing |
| ~1.0% | 6 | `triangular_solve_matrix` / LLT blocked | LLT solve backsubst |
| ~1.0% | 6 | `residualize_genotypes` | covariate residualization |
| ~0.8% | 5 | `general_matrix_vector_product` | OOS GEMV |

### Aggregates (approximate)

| bucket | ≈ self % | notes |
|--------|----------:|-------|
| **GEMM (gebp + pack)** | **~79%** | now the #1 residual hotspot |
| **LLT factor + solve** | **~7%** | cheap vs old eig (~72% baseline) |
| **bed decode** | **~3%** | |
| **residualize + assign + GEMV** | **~4%** | |

`computeFromTridiagonal_impl` / Householder eig path: **absent** from top samples (H1 success).

---

## Fixture sizing note for certification

- Baseline case remains **≥1 s** (formal median 1.950 s) → contract `minimum_case_seconds: 1` satisfied for the *baseline* primary case.
- H1 candidate wall ≈ **0.63 s** (≪1 s). Timing noise is moderate (Wave1 confirm stdev ≈0.012 s on N=8).
- **Recommendation for Wave 2 cert:** prefer **(a) N=30 paired first** under frozen fixture; only grow to fixture v2 if `lower_95` is unreliable near the 2× gate. Residual work after QUALIFIED: GEMM in `calc_cv_matrices` (H4-class) and optional portable `-march=` (H3) — **not** required for H1-only primary claim.

---

## Artifacts

- `research/artifacts/perf_profile_post_H1/perf.data`
- `perf_report_flat.txt`, `perf_report_children.txt`
- `phase_timings.json`, `fit.log`, `wall.txt`
