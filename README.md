# Opti2x optimized regenie v4.1.3

This repository contains an Opti2x-optimized regenie **v4.1.3** source tree.

**Status: QUALIFIED.** The H1 LLT-per-λ optimization measured a **3.135× median speedup** with a **3.054× one-sided 95% lower bound** over **N=30** paired runs.

## Claim scope

The claim is **fixture-scoped**, not package-wide: a serial, Eigen-only Step 1 quantitative-trait (QT) fixture with **N=1000** and **M=2293**, under one-thread controls. The credited change is **H1 LLT-per-λ** in `ridge_level_0`.

This is **not** a package-wide regenie claim, **not** a `-march=native` claim, and **not** an MKL/OpenBLAS claim. H3 (`-march=native`) is deliberately excluded from the QUALIFIED primary result.

## Upstream pin

Upstream: <https://github.com/rgcgithub/regenie>  
Pin: `5ff08954c5ada4c8fd037a8d38cbb57acbf0dd01` (`v4.1.3`)

## Build

Eigen-only build (no MKL/OpenBLAS and no `-march=native`):

```bash
make BGEN_PATH=/path/to/bgen/v1.1.7 HAS_BOOST_IOSTREAM=1 -j"$(nproc)"
```

The packaged source intentionally excludes compiled binaries and object/archive files. See `opti2x/REPRODUCE.md` for the pinned patch, fixture contract, correctness oracle, and timing procedure.

## Evidence

- `opti2x/OPTIMIZATION_REPORT.md` — qualification report and limitations
- `opti2x/CLAIM_SCOPE.md` — explicit claim boundaries
- `opti2x/REPRODUCE.md` — build and certification procedure
- `opti2x/certification.json`, `opti2x/attestation.json`, `opti2x/stats.json` — certification evidence
- `opti2x/ADVERSARIAL_H1.md` — adversarial review (ACCEPT)
