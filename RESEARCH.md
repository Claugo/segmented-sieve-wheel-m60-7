# Research Directions & Open Questions

This document outlines **open research questions** and **potential improvements** 
for the GC-60 Segmented Sieve project.

These are areas where contributions could lead to **novel findings** or **significant optimizations**.

---

## 1. Orthogonal Filter Combinations

### Current State
- Single pre-filter: **p = 7**
- Pre-filter is **manually crafted** as a 7×16 lookup table
- Extends mod-60 wheel to effectively mod(60×7) = mod 420 without structural enlargement

### Open Questions

**Q1: Can we extend to multiple orthogonal filters?**

Currently: mod-60 + p=7 (manual tables)

Hypothesis: We can layer additional filters (p ∈ {11, 13, 17}) orthogonally without:
- Expanding the 16-bit mask
- Degrading cache locality
- Increasing structural complexity

**To Investigate:**
```cpp
// Question: Would this work?
const int SOTTOLISTE_11[11][16] = { ... };  // 11 classes instead of 7
const int SOTTOLISTE_13[13][16] = { ... };  // 13 classes instead of 7

// Can we combine them as separate, independent layers?
```

**Expected Outcome:** 
- Eliminate more composite candidates before sieving
- Reduce overall sieve time
- Potentially reach mod(60×7×11) ≈ mod 4620 range

---

**Q2: What is the optimal filter combination?**

Not all filter choices are equally good:
- p = 7: 7 residue classes, 16 residues per class
- p = 11: 11 residue classes, ? residues per class
- p = 13: 13 residue classes, ? residues per class

**To Investigate:**
- Which primes give best reduction in candidates?
- Is there a mathematical formula to predict optimal combinations?
- Trade-off: More filters = more setup cost vs fewer sieve candidates

**Expected Outcome:**
- Decision framework for choosing pre-filters
- Possibly a generalized algorithm for any p

---

**Q3: Can pre-filter selection be dynamic?**

Currently: p=7 is hardcoded.

**To Investigate:**
- Can we detect optimal filters at runtime based on system specs?
- Can we tune filter combinations per CPU architecture?
- Example: Different filters for L1 vs L2 cache-bound systems?

---

## 2. Cache Locality vs Periodicity Trade-off

### Current State
- Fixed segment size: **32 KB** (matches AMD Ryzen L1 cache)
- Performance: ~211 ms for 10⁹
- Single test machine: AMD Ryzen 7 4800U

### Open Questions

**Q1: Is 32 KB optimal for all CPUs?**

L1 cache sizes vary:
- Intel: typically 32 KB
- AMD: typically 32-64 KB
- ARM: varies (8-64 KB depending on core)

**To Investigate:**
- Benchmark on different CPUs with their native cache sizes
- Does adaptive segment sizing improve throughput?
- Is there a cache line alignment sweet spot?

**Potential Experiment:**
```cpp
// Current: fixed SEG_SIZE = 32768
const u64 SEG_SIZE = std::max(32768ULL, get_l1_cache_size());
```

**Expected Outcome:**
- Portable, CPU-aware segment selection
- Potentially 5-15% speedup on non-Ryzen systems

---

**Q2: Can we leverage hardware prefetching better?**

Modern CPUs have prefetch strategies. Our access pattern:
- Linear reads of uint16_t arrays
- Stride-based writes (L += p for each prime)

**To Investigate:**
- Does changing segment layout (SoA vs AoS) help prefetch?
- Can we align data to cache line boundaries for better prefetch?
- Would prefetch hints (`__builtin_prefetch`) help?

**Potential Optimization:**
```cpp
// Align sieve array to cache line (64 bytes on most systems)
alignas(64) uint16_t sieve[SEG_SIZE];
```

**Expected Outcome:**
- Better hardware prefetch utilization
- Potentially 5-10% throughput improvement

---

**Q3: SIMD-friendly data layout?**

Current: Scalar operations on uint16_t

**To Investigate:**
- Can we restructure data for SIMD (AVX2, AVX-512)?
- Process 4-8 primes in parallel?
- Vectorized bit manipulation?

**Potential Approach:**
```cpp
// SIMD sieving: process multiple candidates simultaneously
// Using __m256i (256-bit = 16 uint16_t values)
__m256i mask1 = _mm256_load_si256((__m256i*)ptr);
__m256i mask2 = _mm256_load_si256((__m256i*)(ptr + 16));
mask1 = _mm256_and_si256(mask1, m);  // Bitwise AND
mask2 = _mm256_and_si256(mask2, m);
_mm256_store_si256((__m256i*)ptr, mask1);
```

**Expected Outcome:**
- 2-4x speedup if SIMD is effective
- But: May lose cache locality benefits

---

## 3. Dual-Level Sieving

### Current State
- Single-pass: all primes together
- No distinction between "small" and "big" primes

### Open Questions

**Q1: Should we separate small and big primes?**

Similar to primesieve's approach:
- **Small primes** (p ≤ √n): Many multiples per segment, use small primes efficiently
- **Big primes** (p > √n): Few multiples, different optimization strategy

**To Investigate:**
- Implement EratSmall (for p ≤ threshold)
- Implement EratBig (for p > threshold) with bucket-based approach
- Measure combined performance

**Expected Outcome:**
- Better cache utilization
- Potentially 20-40% speedup
- Matches primesieve architecture

---

**Q2: What is the optimal threshold?**

Current: No threshold, treat all equally.

**To Investigate:**
- At what prime size does memory access pattern change?
- Is √n the optimal split point?
- Does it vary by CPU?

---

## 4. Theoretical Bounds & Correctness

### Current State
- Empirically validated: Miller-Rabin tests
- No formal mathematical proof of correctness
- Complexity: Claimed O(N log log N) but unproven

### Open Questions

**Q1: Can we prove correctness formally?**

**To Investigate:**
- Prove that GC-60 coordinate system covers all primes
- Prove orthogonal pre-filters don't create gaps
- Formal specification of the sieve invariants

**Potential Output:**
- Mathematical paper proving correctness
- Formal verification (e.g., Coq, Isabelle)

---

**Q2: Is complexity really O(N log log N)?**

**To Investigate:**
- Detailed complexity analysis including:
  - Initialization cost: O(?) for memcpy doubling
  - Sieving loop: O(N log log N) or better?
  - Pre-filter overhead: O(?)
- Compare theoretically vs empirically

**Potential Discovery:**
- Maybe we achieve better than O(N log log N)?
- Or identify bottlenecks preventing it

---

**Q3: Memory lower bounds?**

With fixed 16-bit masks:
- Memory used: ~N/8 bytes (2 bits per candidate, packed)
- Is this optimal?

**To Investigate:**
- Information-theoretic lower bounds
- Can we use fewer than 1 bit per candidate?

---

## 5. Comparison with Related Approaches

### Current State
- Compared informally with primesieve, Atkin sieve
- No published head-to-head benchmark suite

### Open Questions

**Q1: How does GC-60 compare with Sieve of Atkin?**

Atkin claims O(N) time with different approach.

**To Investigate:**
- Implement Atkin sieve
- Benchmark against GC-60 on same hardware
- When is each better?

---

**Q2: Pritchard's Wheel vs GC-60?**

Pritchard's Wheel is a classic wheel factorization.

**To Investigate:**
- Detailed comparison with Pritchard's approach
- Why GC-60 is novel compared to prior wheel work

---

## 6. GPU Acceleration

### Current State
- CPU-only single-threaded
- No GPU support

### Open Questions

**Q1: Can GPUs accelerate the sieve?**

**To Investigate:**
- CUDA implementation for NVIDIA GPUs
- OpenCL for portability
- Does GPU's parallel architecture fit sieving well?

**Challenges:**
- Memory bandwidth vs computation
- Segment coordination across GPU threads
- Data movement overhead

**Potential Outcome:**
- Novel GPU sieve implementation
- Possible 10-100x speedup for massive ranges (10¹²+)

---

## 7. Extended Periodicity Analysis

### Current State
- Orthogonal p=7 pre-filter exists
- Pattern manually coded in SOTTOLISTE array

### Open Questions

**Q1: Can we auto-generate pre-filter tables?**

**To Investigate:**
```cpp
// Generate SOTTOLISTE[p][16] automatically for any prime p
std::vector<std::vector<int>> generate_prefilter(int p) {
    // Algorithm: for each residue r mod 60,
    // check if (60k + 10 + r) is excluded by p
    // Return pattern matrix
}
```

**Expected Outcome:**
- Generalized pre-filter generation
- Easy extensibility to new filters

---

**Q2: Optimal periodicity combinations?**

Some combinations might be better than others:
- 60 × 7 = 420
- 60 × 11 = 660
- 60 × 7 × 11 = 4620

**To Investigate:**
- Metrics: candidate reduction vs memory cost vs computation
- Mathematical optimization framework

---

## 🔬 How to Contribute to Research

### If You're a Researcher/Mathematician
1. Pick a question that interests you
2. Open an Issue with your approach
3. Collaborate with the maintainers
4. Publish findings (with repo attribution)

### If You're a Developer/Engineer
1. Choose an optimization direction
2. Implement and benchmark
3. Submit PR with results
4. Help document the trade-offs

### If You're a Student
1. Great thesis material!
2. Many of these can be Masters/PhD projects
3. Cite the repo, contribute back findings

---

## 📚 Suggested Reading

- Pritchard, P. (1987). "Linear prime-number sieves"
- Atkin, A. O. L. & Bernstein, D. J. (2003). "Prime sieves using binary quadratic forms"
- primesieve documentation and source
- Wheeler's wheel factorization work

---

## 📞 Contact & Discussion

Have ideas on any of these topics?
- Open an Issue with tag `[research]`
- Open a [GitHub Discussion](https://github.com/Claugo/segmented-sieve-wheel-m60-7/discussions) — open and reachable!
- Email maintainer with detailed proposal

---
