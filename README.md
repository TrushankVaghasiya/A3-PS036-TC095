# PQC DNSSEC MTL Benchmark

Benchmarking a post-quantum signature algorithm in **Merkle Tree Ladder (MTL) mode**
for DNSSEC, against equivalent workloads without MTL.

**PS-036**

📺 **Video walkthrough:** _[https://youtu.be/kdxlqXvP3Js]_

---

## The problem

Quantum computers will eventually break the signature algorithms DNSSEC relies on
today. Post-quantum cryptography (PQC) fixes that, but introduces a new constraint:
**signature size**.

| Algorithm | Signature size |
|---|---|
| ECDSA P-256 | 64 B |
| RSA-2048 | 256 B |
| ML-DSA | 2,420 B |
| **SLH-DSA-SHA2-128s** | **7,856 B** |

A DNS response must fit inside roughly **1,232 bytes** before it risks fragmentation
or a TCP fallback. A single SLH-DSA signature is therefore **more than 6× the entire
response**.One signature cannot fit in one datagram.

## The approach

**Merkle Tree Ladder (MTL) mode** (`draft-harvey-cfrg-mtl-mode`) keeps an evolving
forest of perfect binary trees over a message series. Rather than signing every
record individually, the signer periodically signs a small structure called a
*ladder* that covers many records at once, and each record is authenticated with a
compact Me*erkle authentication path.

This means the common case on the wire is a small *condensed signature* with no PQC
signature in it at all; the expensive signature appears only in occasional *reference
values*.

**The catch — and the whole point of this benchmark:** MTL only pays off if records
are signed **in batches**. Sign a fresh ladder per record and you pay a full PQC
signature *plus* hashing overhead, which is strictly worse than not using MTL at all.
Batch size is therefore an experimental variable, not an implementation detail.

## What we measure

Two arms (**with MTL** / **without MTL**) × **three dataset sizes** = six
measurement cells.

| Metric | Definition |
|---|---|
| Signing latency | wall-clock time to produce signatures |
| Verification time | wall-clock time to verify signatures |
| Transmitted bytes | total serialized signature bytes on the wire |

Additional reported quantities:

- **Cache usage** measured directly, with analytical estimates reported separately
- **Batch size (α)** and dataset size (N) recorded for every run

All trials run in a single pinned environment with repeated runs per configuration.

## Our Team

 - [Nithik R](https://links.recu.blog)
 - [Trushank Vaghasiya](https://github.com/TrushankVaghasiya)
 - [Dr. Sumit Mishra](https://erp.nitw.ac.in/ext/profile/cs-sumit)
