# Adversarial review — H1 LLT per λ (regenie v4.1.3)

**Date:** 2026-09-29 PT  
**Candidate:** `experiments/H1_llt_per_lambda/`  
**Binary sha256:** `4d3ecff43a46a079f6e869d22ff81f175db7c72c698a6461d0f074e29f3c2d8d`  
**Pin:** `5ff08954c5ada4c8fd037a8d38cbb57acbf0dd01` (v4.1.3)  
**Patch:** single file `src/Step1_Models.cpp` inside `ridge_level_0` only  
**Verdict: ACCEPT for credit**

---

## 1. Mathematical claim

### Baseline (per fold \(i\))

\[
A_i = GG^\top - G_i G_i^\top = \texttt{ww1},\quad
b_i = GY - G_i^\top Y = \texttt{GTY}-\texttt{GtY[i]}
\]

Eigendecompose \(A_i = U D U^\top\) (symmetric), then for each ridge parameter \(\lambda_j\):

\[
\hat\beta_j = U (D + \lambda_j I)^{-1} U^\top b_i
\]

(Code transforms \(U^\top b_i\) once, then applies the diagonal for each \(\lambda_j\).)

### H1 (per fold \(i\), per \(\lambda_j\))

\[
\hat\beta_j = (A_i + \lambda_j I)^{-1} b_i
\quad\text{via Eigen }\texttt{LLT}\texttt{.solve}
\]

### When are they identical?

**Exact arithmetic:** For any symmetric \(A_i\) and any \(\lambda_j\) with \(D+\lambda_j I\) invertible,

\[
U(D+\lambda_j I)^{-1}U^\top = (A_i + \lambda_j I)^{-1}.
\]

So the *maps* are algebraically the same ridge estimator. No LOCO formula change, no λ-grid change, no fold change.

**Floating point:** Eigendecomposition + diagonal inverse and Cholesky solve take different roundoff paths. In general they are **not** bit-identical. Byte-identical LOCO on a given fixture is a strong empirical outcome, not a theorem.

**SPD / LLT applicability:** Default l0 fractions `[0.01, 0.25, 0.5, 0.75, 0.99]` are remapped by

\[
\lambda \leftarrow M\cdot(1-\lambda_{\mathrm{frac}})/\lambda_{\mathrm{frac}}
\]

(`Data.cpp`), yielding strictly positive λ on this fixture (≈ `[227007, 6879, 2293, 764.3, 23.16]` for \(M=2293\)). For PSD Gram \(A_i\), \(A_i+\lambda I\) is SPD for \(\lambda>0\), so LLT is valid. If a user set \(\lambda=0\) or a singular \(A_i\) with tiny λ, LLT could fail where the eig path with \((D+\lambda)^{-1}\) still runs — **out of scope for default grids; documented residual risk**.

H1 uses **LLT**, not LDLT (patch audit).

---

## 2. Cheat / regression checklist

| Check | Result |
|-------|--------|
| Skipped LOCO / changed write path | **No** — patch only replaces β solve inside `ridge_level_0`; LOCO/`write_predictions` untouched |
| Changed λ grid | **No** — same `params->lambda(j)`; log still `ridge data_l0 : [ 5 : 0.01 0.25 0.5 0.75 0.99 ]` |
| Changed CV folds / LOOCV swap | **No** — still 5-fold; `ridge_level_0_loocv` untouched; `use_loocv` unchanged |
| Float mode / `-ffast-math` / ISA | **No** — same Makefile `CXXFLAGS` as baseline (`-O3 -ffast-math …`); **no** `-march=native` in H1-only |
| Thread bumps / OpenMP | **No** — still `--threads 1` + env caps; NLWP probe max=1 in Wave1 and stress |
| MKL / OpenBLAS switch | **No** — Eigen-only; patch has no `WITH_MKL`/`WITH_OPENBLAS` |
| Oracle edits | **No** — comparator unchanged; candidate matches frozen oracle |
| Undefined behavior | **No evidence** — LLT on SPD \(A+\lambda I\); no unchecked `.info()` but λ≫0 on defaults; exit 0 all stress runs |
| Scope creep | **Single hunk** in `Step1_Models.cpp`; forbidden-token scan hits on `n_ridge`/`cv_folds` are **context lines**, not edits |

---

## 3. Stress evidence

Artifacts under `research/adversarial_H1/` (`run_stress.sh`, `stress.log`, `stress_summary.txt`).

| Stress | Result |
|--------|--------|
| Dev fixture vs frozen oracle ×3 | **PASS** — `fit_1.loco` sha `a6407f4a…b13c` all three; pred.list basename-normalized match; walls ≈0.63–0.67 s |
| `example/` QT smoke: H1 vs baseline | **PASS** — loco **byte-identical** (`a65ad4f3…bd7f`); pred.list basename match |
| Alt-seed cheap fixture (N=400, M=736, seed=20260930): H1 vs baseline | **PASS** — loco **byte-identical** (`d879532c…ddf6`); pred.list match |
| Patch file set | Only `src/Step1_Models.cpp` |

Unexpected but repeatedly confirmed: on these QT step1 paths, H1 LOCO is **byte-identical** to the eig path (frozen oracle and live baseline). That exceeds the algebraic claim and closes the Wave1 “oracle risk” concern for this claim_scope.

---

## 4. What H1 does *not* claim

- Not portable ISA / `-march=native` (that is H3; **not** required for H1-only QUALIFIED).
- Not package-wide 2×; fixture-scoped only.
- Not LOOCV path; not BT/Cox special cases in this review (fixture is QT K-fold).
- Not that LLT equals eig bit-for-bit for all λ/user configs — only that the estimator is the same and that observed primary outputs match on exercised QT step1 inputs.

---

## 5. Verdict

**ACCEPT for credit** as the primary algorithmic optimization for a portable Eigen-only QUALIFIED stack.

Rationale: mathematically equivalent ridge solve for λ>0; no forbidden lever; oracle byte-identical on development fixture (×3), example smoke, and an alternate-seed fixture; NLWP=1; H1-only (no host-ISA flag).
