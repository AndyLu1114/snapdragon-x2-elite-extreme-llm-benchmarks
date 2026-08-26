# Snapdragon X2 Elite Extreme: LLM Inference Benchmarks

This repository documents LLM inference benchmarks executed on the Snapdragon X2 Elite Extreme (Adreno X2-90) using `llama.cpp`. 

For comprehensive hardware details, please refer to the [Snapdragon X2 Elite Extreme Device Specification](docs/device-spec.md).

### Theoretical Decoding Limits

For a Qwen 27B model utilizing Q4 quantization, the theoretical maximum decode speed is fundamentally memory-bound. It can be calculated as follows:

$$
\text{Max Decode Speed} = \frac{\text{System Bandwidth}}{\text{Model Size in Bytes}}
$$

$$
\text{Max Decode Speed} = \frac{152 \text{ GB/s}}{16.06 \text{ GB}} \approx 9.46 \text{ tokens/s}
$$

*Note: Real-world performance will be slightly lower due to system overhead and KV cache management.*

---

## 1. Qwen 27B (4-bit Quantization) Without MTP

**Model Overview:** Qwen3.6-27B is an open-weight, 27-billion-parameter multimodal model released by Alibaba in April 2026. It features a dense architecture where all parameters are active during inference, native support for a 262K-token context window.

### CPU vs. GPU Performance

![Qwen 27B Without MTP](/plots/QWEN_27b_no_mtp_CPU_GPU.png "no mtp QWEN 27B")

Our initial baseline comparison evaluates CPU and GPU performance without Multi Token Prediction (MTP) enabled.

* **Cold Start Advantage:** From a cold start, the CPU achieves superior decode and prefill speeds compared to the GPU.
  * CPU Decode: 6.69 tokens/sec.
  * CPU Prefill: 7.87 tokens/sec.
* **Thermal Throttling:** As the CPU warms up under sustained load, its performance degrades, allowing the GPU to surpass it in decode throughput.
* **Stability:** The GPU demonstrates highly stable performance across consecutive runs.

### Configuration & Results

**CPU Test Environment**
Server (Terminal 1):

```powershell
$M = .\Qwen3.6-27B-MTP-GGUF-Q-4-0\Qwen3.6-27B-Q4_0.gguf
.\llama-server -m $M -ngl 0 -c 4096 --port 8080
```

Client (Terminal 2):

```powershell
$body = @{ prompt = "test"; n_predict = 256; temperature = 0; ignore_eos = $true } | ConvertTo-Json
```

**GPU Test Environment**
Server (Terminal 1):

```powershell
.\llama-server -m $M -ngl 99 -c 4096 --port 8080
```

Client (Terminal 2):

```powershell
$body = @{ prompt = "test"; n_predict = 256; temperature = 0; ignore_eos = $true } | ConvertTo-Json
```

**Benchmark Results (Tokens/sec)**

| Backend                    | Run 0 (Cold) | Run 1       | Run 2       |
|:-------------------------- |:------------ |:----------- |:----------- |
| **CPU (Decode / Prefill)** | 6.69 / 7.87  | 5.20 / 5.20 | 5.28 / 5.20 |
| **GPU (Decode / Prefill)** | 6.11 / 6.00  | 6.11 / 5.90 | 6.10 / 6.00 |

---

## 2. Qwen 27B (4-bit Quantization) With MTP

Multi Token Prediction (MTP) is a speculative decoding technique that enables models like Qwen3.6 to achieve approximately 1.4x to 2.2x faster generation without sacrificing accuracy. According to Unsloth, this provides the Qwen3.6 27B and 35B-A3B models a >1.4x speed-up over their baselines, making it a highly advantageous feature for local, on-device deployments.

### CPU vs. GPU Performance with MTP

![Qwen 27B CPU vs GPU with MTP](\plots\QWEN_27b_mtp_CPU_GPU.png "QWEN 27B CPU GPU benchmark")

While Unsloth notes that `--spec-draft-n-max 2` works best for most general setups (and recommends testing values from 1 to 6), hardware-specific testing reveals a different optimal configuration for this ARM architecture.

* **CPU Optimization:** On the Snapdragon X2 Elite Extreme device, the CPU achieves peak performance with an `n-max` of 3.
  * Decode: 16.85 tokens/sec.

* **GPU Limitations:** The GPU gradually improves its throughput as `n-max` scales from 2 to 6, but overall, it performs substantially worse when MTP is enabled.
* **Technical Bottleneck:** This GPU performance degradation is likely due to MTP forcing `ne11 > 1`, which routes operations to a tiled GEMM. Because a dedicated small-batch kernel does not currently exist, there is a noticeable performance gap between the highly optimized batch-1 specializations and the large-batch GEMMs utilized during MTP.

### Configuration

**CPU Test Environment (MTP enabled, n-max 2)**
Server (Terminal 1):

```powershell
$M = .\Qwen3.6-27B-MTP-GGUF-Q-4-0\Qwen3.6-27B-Q4_0.gguf
.\llama-server -m $M -ngl 0 -c 4096 --spec-type draft-mtp --spec-draft-n-max 2 --port 8080
```

Client (Terminal 2):

```powershell
$body = @{ prompt = "test"; n_predict = 256; temperature = 0; ignore_eos = $true } | ConvertTo-Json
```

**GPU Test Environment (MTP enabled, n-max 2)**
Server (Terminal 1):

```powershell
.\llama-server -m $M -ngl 99 -c 4096 --spec-type draft-mtp --spec-draft-n-max 2 --port 8080
```

Client (Terminal 2):

```powershell
$body = @{ prompt = "test"; n_predict = 256; temperature = 0; ignore_eos = $true } | ConvertTo-Json
```
