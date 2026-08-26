# Device Specification

Test platform for all benchmarks in this repository.

|                 |                             |
| --------------- | --------------------------- |
| **SoC**         | Snapdragon X2 Elite Extreme |
| **Part number** | X2E94100                    |
| **OS**          | Windows 11 on Arm (Arm64)   |

---

## CPU

| Property     | Value                    |
| ------------ | ------------------------ |
| Architecture | Qualcomm Oryon (3rd gen) |
| Cores        | 18                       |
| Threads      | 18 (no SMT)              |
| Max clock    | 4.7 GHz                  |
| Base clock   | 4.45 GHz                 |
| L1 cache     | 4.5 MB                   |
| L2 cache     | 44.0 MB                  |

**ISA features reported by llama.cpp:**

```
NEON = 1 | ARM_FMA = 1 | MATMUL_INT8 = 1 | DOTPROD = 1 | OPENMP = 1 | REPACK = 1
```

Not available: `SVE`, `SME`, `FP16_VECTOR_ARITHMETIC` — these failed detection during the llama.cpp build.

`MATMUL_INT8` and `DOTPROD` matter for quantized inference: they enable the accelerated integer GEMM path, and `REPACK` reorders Q4_0 weights into a layout that path can use.

---

## GPU

| Property              | Value                   |
| --------------------- | ----------------------- |
| Model                 | Qualcomm Adreno X2-90   |
| Clock                 | 1.85 GHz                |
| Compute API           | OpenCL 3.0              |
| Driver version        | 32.0.149.0 (2026-02-06) |
| Max OpenCL allocation | 24,379 MiB              |

As reported by `llama-completion --list-devices`:

```
GPUOpenCL: Qualcomm(R) Adreno(TM) X2-90 GPU (24379 MiB, 23355 MiB free)
```

---

## NPU

| Property    | Value            |
| ----------- | ---------------- |
| Model       | Qualcomm Hexagon |
| Performance | 80 TOPS          |

Not used in these benchmarks. The llama.cpp Hexagon backend requires a separate build, the Hexagon SDK, and Windows test-signing mode.

---

## Memory

| Property       | Value        |
| -------------- | ------------ |
| Capacity       | 48 GB        |
| Type           | LPDDR5X      |
| Transfer rate  | 9600 MT/s    |
| Peak bandwidth | 152 GB/s     |
| Memory slots   | 0 (soldered) |

---

## Storage

| Property  | Value         |
| --------- | ------------- |
| Interface | PCIe 4.0 NVMe |

Relevant only to model load time. All benchmarks use `--no-mmap` so weights are fully resident in RAM before timing begins.

---

## Sources

- CPU / GPU / NPU / memory specifications — [Qualcomm Snapdragon X2 Elite product page](https://www.qualcomm.com/laptops/products/snapdragon-x2-elite)
- Memory transfer rate — retailer listing (Best Buy)
- Cache sizes, base clock, driver version — Windows Task Manager
- ISA feature flags, OpenCL device string — llama.cpp runtime output
