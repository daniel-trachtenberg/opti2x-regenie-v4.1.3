# Reproduce — regenie v4.1.3 Opti2x QUALIFIED (H1-only)

Contract: `regenie-v4.1.3-serial-eigen-step1-qt-n1000-m2293-v1`  
Pin: `5ff08954c5ada4c8fd037a8d38cbb57acbf0dd01` (`v4.1.3`)  
Candidate binary sha256: `4d3ecff43a46a079f6e869d22ff81f175db7c72c698a6461d0f074e29f3c2d8d`

## 0. Worktree already built (this run)

```text
repository root (./)
```

The compiled candidate binary is intentionally not committed; its expected SHA-256 is recorded in `opti2x/regenie.h1.sha256`.

## 1. Clone and checkout pin

```bash
git clone https://github.com/rgcgithub/regenie.git
cd regenie
git checkout 5ff08954c5ada4c8fd037a8d38cbb57acbf0dd01
```

## 2. Apply patch

```bash
patch -p1 < opti2x/optimized.patch
# sha256: bed024569927e08dc815049450f31f63c86ba5c42c3c0fc65b7cfe19e1b0986f
```

Fairness patch: **none**.

## 3. Dependencies + build

External BGEN v1.1.7 required (see `locks/BGEN.txt` / `deps/bgen/v1.1.7`). Eigen is vendored. **Do not** set `MKLROOT` / `OPENBLAS_ROOT`. **Do not** add `-march=native` for this QUALIFIED binary.

```bash
BGEN_PATH=/path/to/bgen/v1.1.7 HAS_BOOST_IOSTREAM=1 make -j$(nproc)
sha256sum regenie
# expected on this host/toolchain: 4d3ecff43a46a079f6e869d22ff81f175db7c72c698a6461d0f074e29f3c2d8d
```

Baseline (unpatched pin, same flags): `4bd69197dead7415860b4e066cffb0d7a974fa3f655076fd130921131383d3d7`.

## 4. Fixture

The qualification fixture and harness are not included in this source-only package. Their contract, hashes, and captured timing evidence are preserved in `opti2x/certification.json`, `opti2x/certification_timing.json`, and `opti2x/stats.json`.


```text
benchmarks/dev_fixture_step1_qt/inputs/regenie_dev_n1000.{bed,bim,fam,pheno,covar}
```

Generator: `benchmarks/dev_fixture_step1_qt/scripts/generate_fixture.sh` (seed `20260929`).  
Hashes in `benchmarks/development.yaml`.

## 5. Run argv (serial envelope)

```bash
export OMP_NUM_THREADS=1 MKL_NUM_THREADS=1 OPENBLAS_NUM_THREADS=1 GOTO_NUM_THREADS=1
taskset -c 0 ./regenie --step 1 \
  --bed benchmarks/dev_fixture_step1_qt/inputs/regenie_dev_n1000 \
  --covarFile benchmarks/dev_fixture_step1_qt/inputs/regenie_dev_n1000.covar \
  --phenoFile benchmarks/dev_fixture_step1_qt/inputs/regenie_dev_n1000.pheno \
  --bsize 500 --threads 1 \
  --lowmem --lowmem-prefix /tmp/rg_l0 \
  --out /tmp/fit
```

## 6. Oracle

```bash
sha256sum /tmp/fit_1.loco
# expect: a6407f4af47428dfb556f7c82edccd7c7ea81962dad9ca4806758e32fdb7b13c
# pred.list: basename-normalize loco path field; compare to oracle/fit_pred.list
```

## 7. Paired timing

```bash
# From run root:
./experiments/_helpers/cert_paired_H1.sh 30 20260929
# Captured result is preserved in opti2x/stats.json
```

Expect median speedup ≈3.13× and lower_95 ≈3.05× on a quiet machine comparable to this host; absolute walls vary by CPU.
