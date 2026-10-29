# pyasn1-rust

Experimental Rust port lane for [pyasn1](https://github.com/pyasn1/pyasn1)
(BER/CER/DER codec). BSD-2-Clause, per the upstream license; see
`NOTICE` for attribution.

## Lane outcome (2026-10-29): `no_gain` — no Rust port scoped

Tier-1 pure-Python fixes measured first on a frozen 3000-record
BER roundtrip workload (Ryzen 9 5950X, CPython 3.12.3, pyasn1 0.6.5,
min of 5, wall + peak RSS both arms):

| arm | representative | vs A |
|---|---|---|
| A stock | 0.2694 s | 1.00x |
| B decoder-instance reuse (tag caches warm) | 0.2401 s | 1.12x |
| C bytes.join assembly | 0.2758 s | 0.98x |
| D header memoization | 0.2851 s | 0.94x |
| E combined | 0.2307 s | **1.17x** |

Kill math against the tier-1 baseline: the object-model floor
(building the same records from native values, 0.0508 s) leaves a
decode-infinite ceiling of **1.83x < 2x** (realistic ≈1.4–1.6x);
with warm caches, `type/*` construction (33%) exceeds remaining
decoder dispatch, and freeing tag/header assembly measures ~1.0x.
A whole-message Rust framing kernel keeping the Python object model
cannot clear the bar without per-field crossings or reimplementing
the object model. Honest `no_gain`.

The upstream-ready tier-1 patch (shared decoder tag caches + join
assembly, 1.14x through the public API, full upstream suite
1261 passed, 26/26 equivalence) is proposed in the lane record, not
opened as an upstream PR. Evidence, ADR, and bench detail live in
the rustsmith evidence PR (tooling/evidence/ADRs only, per lane
rules) — this repo holds license/notice/outcome only.
