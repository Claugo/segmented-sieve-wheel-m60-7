# Contributing to GC-60 Segmented Sieve

Thank you for your interest in contributing! This document outlines how you can help improve this project.

---

## 🎯 Areas We Need Help With

### Performance & Optimization
- Profiling on different CPU architectures (Intel, ARM, AMD)
- Cross-platform testing (Windows, Linux, macOS)
- SIMD/AVX2 optimizations for the sieve loop
- Cache behavior measurement and tuning
- Benchmark suite vs primesieve

### Extensions & Research
- Implementing additional orthogonal pre-filters (p ∈ {11, 13, 17, ...})
- Dual-level sieving (separate handling for small/big primes)
- Multi-threading strategies with atomic coordination
- GPU acceleration exploration
- Mathematical proofs and correctness validation

### Documentation & Academic Work
- Academic paper drafts explaining the GC-60 model
- Detailed algorithm walkthroughs
- Visual diagrams and illustrations
- Mathematical notation improvements
- Educational blog post ideas

### Testing & Quality
- Unit tests for critical functions
- Correctness validation on different ranges
- Regression test suite
- Coverage analysis

---

## 🚀 How to Contribute

### Step 1: Fork & Clone
```bash
git clone https://github.com/YOUR_USERNAME/segmented-sieve-wheel-m60-7.git
cd segmented-sieve-wheel-m60-7
```

### Step 2: Create a Feature Branch
```bash
git checkout -b feature/your-improvement
# Example: feature/avx2-vectorization
```

### Step 3: Make Your Changes
- Keep commits **focused** and **descriptive**
- Write clear commit messages
- If adding code, include explanatory comments

### Step 4: Test & Benchmark
```bash
# Build and test
# Ensure your changes don't break existing functionality
./sieve_wheel_M60_7 1000000000
```

### Step 5: Submit a Pull Request (PR)
- Give a **clear title**: "Add AVX2 optimization" or "Implement p=11 prefilter"
- In the **description**, include:
  - What problem does it solve?
  - What changed?
  - Any performance measurements (before/after)?
  - Any new dependencies?

---

## 💭 Before You Start

**Please open an Issue first** for:
- Major architectural changes
- New dependencies
- Significant refactoring

This lets us discuss the approach before you invest time coding.

**Example Issue:**
```
Title: "Request: Add multi-threading support"

Description:
I'd like to implement parallel segment processing using std::thread.
The idea is to divide segments among threads with atomic updates.

Is this aligned with the project direction?
```

---

## 📋 Code Style Guidelines

### C++ Standards
- Use **C++17 standard** minimum
- Compile with: `-std=c++17 -O3 -march=native`

### Naming Conventions
- **Variables**: `snake_case` (preferred) or Italian names with English comments
- **Functions**: `snake_case`
- **Structs**: `PascalCase`
- **Constants**: `UPPERCASE`

### Comments
- Use **English for code comments**
- Variable names can be Italian (but add English descriptions)
- Explain the *why*, not just the *what*

### Example:
```cpp
// Good: Explains the logic
u64 limite = (u64)std::sqrt(CERCA_IN + 60);  // Upper bound for prime generation

// Avoid: Just repeats code
u64 limite = (u64)std::sqrt(CERCA_IN + 60);  // Calculate limite
```

---

## 🔬 Performance Contributions

If you're optimizing code:

1. **Measure baseline first**
   ```bash
   time ./sieve_wheel_M60_7 1000000000
   ```

2. **Make one change at a time**

3. **Measure again** and report:
   ```
   Before: 211 ms
   After:  185 ms
   Improvement: ~12%
   ```

4. **Document** what changed and why

---

## 📚 Testing Your Changes

### Basic Correctness Test
```bash
./sieve_wheel_M60_7 1000000000
# Should output:
# - Correct candidate density
# - All Miller-Rabin tests pass (100/100 OK)
```

### Larger Range Test
```bash
./sieve_wheel_M60_7 10000000000
# Monitor execution time and memory usage
```

---

## 🤝 Code Review Process

1. You submit a PR
2. We review the code for:
   - Correctness
   - Performance impact
   - Code style adherence
   - Documentation quality
3. We may request changes
4. Once approved, we merge! 🎉

---

## ❓ Questions?

- **Have an idea?** Open an Issue first
- **Found a bug?** Report it with details (CPU, OS, range tested)
- **Want to discuss?** Check GitHub Discussions (when enabled)

---

## 📄 License Agreement

By contributing, you agree that your contributions will be licensed under the **MIT License** (same as the project).

---

## 🙏 Thank You!

Contributors are the heart of open source. We appreciate your time and effort!

---
