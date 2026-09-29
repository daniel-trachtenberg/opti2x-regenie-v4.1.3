# Opti2x Wave 2 report — regenie v4.1.3

## Terminal status: **QUALIFIED**

**Contract:** `regenie-v4.1.3-serial-eigen-step1-qt-n1000-m2293-v1`  
**Fixture revision:** 1  
**Candidate:** `experiments/H1_llt_per_lambda/` (H1-only)  
**Pushable worktree:** `repository root (./)`  
**When packaged (PT):** 2026-09-29 ~1:40 PM PT  

Fixture-scoped claim only. See `CLAIM_SCOPE.md`.  
**H3 (`-march=native`) is not credited** in this deliverable.

---

## Speedup + CI (N=30 paired vs frozen baseline)

Evidence: `stats.json` and `certification_H1.json`  
Harness: `experiments/_helpers/cert_paired_H1.sh` (warmup + NLWP probe + 30 randomized pairs, seed 20260929).

| Metric | Value |
|--------|------:|
| Parent (baseline) median wall | 1.975 s |
| Child (H1) median wall | 0.630 s |
| **Median speedup (medians)** | **3.135×** |
| Median paired speedup | 3.102× |
| Child faster | **30/30** |
| **One-sided 95% Student-t lower bound** | **3.054×** |
| Estimator | one-sided Student-t 95% LB on mean(log(parent/child)) |

Gates: median ≥ 2.0 ✓, lower_95 ≥ 2.0 ✓, oracle PASS ✓, NLWP=1 ✓, adversarial ACCEPT ✓.

Formal frozen baseline median (prior N=30 solo): **1.950 s** (baseline recorded in the certification evidence). Paired parent median 1.975 s is consistent.

---

## Composition (credited)

| Layer | Hypothesis | Role |
|-------|------------|------|
| Wave1 / Wave2 | **H1** LLT per λ | Replace fold eigendecomp with Cholesky solves of `(ww1+λI)` |

**Not credited:** H3 `-march=native`; H1+H3 stack; H2/H4/H15.

**Build:** Eigen-only `make` with `HAS_BOOST_IOSTREAM=1`, locked BGEN v1.1.7. Same CXXFLAGS as baseline (no extra `-march`).

---

## Adversarial review

Path: `ADVERSARIAL_H1.md` — **ACCEPT for credit**.

- Algebraically equivalent ridge estimator for λ>0.
- Stress: dev oracle ×3 PASS; example/ QT cross-binary byte-identical; alt-seed (N=400,M=736,seed=20260930) byte-identical.
- No LOCO skip, λ-grid change, fold change, thread bump, MKL switch, or float-mode change.

---

## Post-H1 profile

Path: `PROFILE_post_H1.md`. Residual hotspots: Eigen GEMM (`gebp`) ~75%+; LLT ~7%; bed decode ~3%. L0 ridge log time 1670→401 ms. Fixture not grown: lower_95 stable at ~3.05× with candidate wall ≈0.63 s.

---

## Rejected / not primary (Wave1 ledger)

| ID | Decision | Why |
|----|----------|-----|
| H2 Woodbury | REJECT | ~1.0× |
| H3 `-march=native` | ACCEPT but **not primary** | host-ISA; portable freeze not done |
| H4 rankUpdate Gram | marginal/reject | ~1.02–1.04× |
| H15 prealloc | REJECT | ~1.0× |
| H1+H3 stack | measured ~8.5× | excluded from portable QUALIFIED primary |

Ledger excerpt: `hypotheses_ledger_excerpt.jsonl`. Full notes: the run workspace.

---

## Limitations (explicit)

1. **Fixture-scoped only** — not package-wide regenie 2×.
2. **H1 candidate ≪1 s** while baseline ≥1 s; cert kept N=30 / fixture v1 because lower_95 ≫ 2.0.
3. **NLWP=1 observed** (taskset+env); cgroup-v2 `pids.max` not attested on this host.
4. **Agent-attested** packaging on this host.
5. **Default λ>0** assumed for LLT; LOOCV/BT/Cox not in this cert.
6. **H3 not included** — do not market native-ISA stack as this QUALIFIED artifact.

---

## Hashes

| Artifact | sha256 |
|----------|--------|
| Candidate binary (H1) | `4d3ecff43a46a079f6e869d22ff81f175db7c72c698a6461d0f074e29f3c2d8d` |
| Baseline binary | `4bd69197dead7415860b4e066cffb0d7a974fa3f655076fd130921131383d3d7` |
| `optimized.patch` | `bed024569927e08dc815049450f31f63c86ba5c42c3c0fc65b7cfe19e1b0986f` |
| Oracle `fit_1.loco` | `a6407f4af47428dfb556f7c82edccd7c7ea81962dad9ca4806758e32fdb7b13c` |
| Pin commit | `5ff08954c5ada4c8fd037a8d38cbb57acbf0dd01` |
