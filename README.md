# Cache Performance Evaluation Report

**Course**: CSCE 231 / 2303 – Computer Organization  
**Semester**: Spring 2025  
**Project**: Project 2  
**Student**: Seba Ahmed Wahba – 900225470  
**Section**: 10:00 AM WU  
**Instructor**: Dr. Mohammed Shalan  

---

## 1. Introduction

A cache is essentially a memory storage space introduced to tackle a major performance bottleneck that appeared in the late 1980s. Processors became faster than main memory, making memory a limiting factor. The solution was to add a cache — small, fast SRAM between the CPU and large, slow DRAM — to improve performance by reducing the average memory access time.

When data is found in the cache, it's a **hit**; otherwise, it's a **miss**, which incurs a **miss penalty**.

---

## 2. Cache Simulator Design

**Data structures**:  
- `CacheLine`  
- `Cache` class  

**Replacement Policy**:  
- **LRU (Least Recently Used)**, using a counter bit to track recency.

**Cache Parameters**:  
- **Cache size**: 64 KB  
- **Line size**: 16, 32, 64, 128 bytes  
- **Associativity**: 1, 2, 4, 8, 16 ways  

**Indexing**:  
- Offset = log₂(line size)  
- Index = log₂(number of sets), obtained by shifting and masking

### Memory Generators:

| Generator | Type | Description |
|----------|------|-------------|
| `memGen1` | Sequential | High spatial locality |
| `memGen2` | Random (24 KB region) | High temporal locality |
| `memGen3` | Random (64 MB) | No locality |
| `memGen4` | Sequential (4 KB) | Very high spatial + temporal locality |
| `memGen5` | Sequential (64 KB) | Medium spatial + temporal locality |
| `memGen6` | Strided, skips 32 bytes | Breaks spatial locality for small line sizes |

---

## 3. Experiment Setup

- **1,000,000** memory accesses per test  
- **6 memory generators**  
- **2 Experiments**:  
  1. Fixed 4 sets, vary line size  
  2. Fixed line size (64 bytes), vary associativity  

**Tools**:  
- C++ for simulation  
- Google Sheets for data visualization  

---

## 4. Results & Graphs

### Experiment 1: Hit Ratio vs Line Size (Sets = 4)

| Generator | Line Size (bytes) | Hit Ratio (%) |
|-----------|-------------------|---------------|
| memGen1   | 16                | 93.75         |
|           | 32                | 96.88         |
|           | 64                | 98.44         |
|           | 128               | 99.22         |
| memGen2   | 16                | 99.85         |
|           | 32                | 99.92         |
|           | 64                | 99.96         |
|           | 128               | 99.98         |
| memGen3   | All               | 0.1           |
| memGen4   | 16–128            | ~100          |
| memGen5   | 16–128            | ~99.6–99.95   |
| memGen6   | 16                | 0             |
|           | 32                | 0             |
|           | 64                | 50            |
|           | 128               | 75            |

---

### Experiment 2: Hit Ratio vs Associativity (Line Size = 64)

| Generator | Associativity (ways) | Hit Ratio (%) |
|-----------|----------------------|---------------|
| memGen1–2 | All                  | Constant      |
| memGen3   | All                  | 0.1           |
| memGen4–5 | All                  | ~99.9         |
| memGen6   | All                  | 50            |

---

## 5. Analysis & Discussion

- **Line size effect**: Larger lines helped when spatial locality was present (memGen1, 4, 5).  
- **Associativity**: Little impact overall; most gains came from line size.  
- **Best gains**: Seen in memGen1, 5, 6 with larger line sizes.  
- **Flat performance**: memGen3 saw no benefit regardless of configuration.  
- **Unexpected result**: Associativity didn’t improve memGen6 despite expectations.

---

## 6. Conclusion

- **Line size** has a **major impact** for spatial locality.  
- **Associativity** has **minimal effect** in most realistic scenarios.  
- **Access pattern** understanding is **crucial** to optimizing cache design.  
- **Strided patterns** only benefit if line size is large enough to cover the stride.

---
