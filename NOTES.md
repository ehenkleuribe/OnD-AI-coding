# NOTES — Technical Justification & Authoritative References

This document provides the technical rationale for all design decisions in this setup. Each decision is anchored in official documentation and community-tested best practices.

---

## 1. Vulkan Backend Selection

### Decision
Force Vulkan as the inference backend for both Ollama and LM Studio.

### Rationale
- **Hardware Alignment**: Ryzen AI iGPU has mature Vulkan driver support via AMD's RADV/AMDVLK stack
- **Performance**: Vulkan provides lower-overhead GPU access compared to DirectML on AMD hardware
- **Cross-Platform**: Vulkan ICDs are consistent across Windows, Linux, and future AMD driver updates

### Authoritative Sources
- [llama.cpp Vulkan Backend](https://github.com/ggerganov/llama.cpp/blob/master/docs/build.md#vulkan) — Vulkan build instructions and compatibility notes
- [AMD Vulkan Driver Documentation](https://gpuopen.com/learn/vulkan/) — AMD's official Vulkan driver architecture
- [Ollama Environment Variables](https://github.com/ollama/ollama/blob/main/docs/env.md) — `OLLAMA_LLM_LIBRARY` documentation
- [LM Studio Vulkan Support](https://lmstudio.ai/docs/app/advanced/parallel-requests) — LM Studio hardware backend configuration

### Implementation
```powershell
[System.Environment]::SetEnvironmentVariable('OLLAMA_LLM_LIBRARY', 'vulkan', 'Machine')
```

---

## 2. Single-Active-Model Memory Policy

### Decision
Enforce exactly one model resident in memory at any time (`single_resident: true`).

### Rationale
- **32GB System Constraint**: A 35B parameter model at `q4_k_m` quantization requires ~18-20GB VRAM/RAM. Adding a 4B model plus context overhead exceeds safe memory thresholds
- **System Stability**: Prevents Windows memory paging, which causes inference stalls and potential crashes
- **Inference Quality**: Ensures full model weights remain in fast memory, avoiding degraded token generation

### Memory Budget Breakdown (32GB System)

| Component | Size |
|:---|:---|
| Windows 11 OS | ~4-6 GB |
| Ryzen iGPU Shared VRAM | ~4 GB (reserved) |
| Qwen 35B (q4_k_m) | ~18-20 GB |
| Context Buffer (32k) | ~2-3 GB |
| **Total Required** | **~28-31 GB** |

Loading two models simultaneously would exceed the 32GB envelope, triggering system paging.

### Authoritative Sources
- [llama.cpp Memory Management](https://github.com/ggerganov/llama.cpp/blob/master/docs/memory.md) — VRAM/RAM allocation strategies
- [AMD Ryzen AI Memory Architecture](https://www.amd.com/en/developer/ryzenai.html) — NPU/iGPU memory sharing on Windows
- [Windows 11 Memory Management](https://learn.microsoft.com/en-us/windows/win32/memory/memory-management) — System paging behavior

---

## 3. Context Window & Concurrency Limits

### Decision
`context_window: 32768`, `concurrency: 1`

### Rationale
- **32k Context**: Provides sufficient token depth for code generation while fitting within the 32GB memory envelope when combined with model weights
- **Concurrency 1**: Multi-threaded inference on shared memory systems causes memory fragmentation and context corruption on 32GB rigs

### Authoritative Sources
- [Ollama Context Configuration](https://github.com/ollama/ollama/blob/main/docs/modelfile.md#parameter) — `num_ctx` parameter documentation
- [LM Studio Parallel Requests](https://lmstudio.ai/docs/app/advanced/parallel-requests) — Concurrency limits and memory impact

---

## 4. Model Selection Rationale

### Build Model — Gemma 4B

| Attribute | Value |
|:---|:---|
| Parameters | 4B |
| Quantization | Default (Q4_K_M equivalent) |
| VRAM Footprint | ~2.5-3 GB |
| Use Case | Rapid code iteration, syntax correction, boilerplate generation |

**Why 4B**: Fits comfortably in memory after Architect model unloads, enabling fast feedback loops without memory pressure.

**Authoritative Source**: [Google Gemma Models](https://ai.google.dev/gemma) — Official model specifications

### Architect Model — Qwen 3.6 35B

| Attribute | Value |
|:---|:---|
| Parameters | 35B |
| Quantization | `q4_k_m` |
| VRAM Footprint | ~18-20 GB |
| Use Case | Deep reasoning, architecture planning, complex code refactoring |

**Why Qwen 3.6**: Strong reasoning capabilities at 35B parameters, with `q4_k_m` quantization providing the best quality-to-size ratio for 32GB systems.

**Authoritative Source**: [Qwen Model Family](https://github.com/QwenLM/Qwen) — Official model documentation and quantization recommendations

---

## 5. Vulkan ICD Troubleshooting (Conditional)

> **Note**: The following is provided for cases where the Vulkan ICD is not detected by the inference engine.

### Verify Vulkan ICD Registration

```powershell
Get-ChildItem -Path "C:\Windows\System32\vk_icd*.json" | Select-Object Name, Length
```

Expected: `vk_icd_amd64.json` or similar AMD Vulkan ICD file.

### Verify Vulkan Runtime

```powershell
lms info --gpu
```

Expected output should list Vulkan as the active backend with GPU compute units detected.

### Reinstall AMD Vulkan Runtime

1. Download [AMD Vulkan Runtime](https://www.amd.com/en/support)
2. Run installer as Administrator
3. Reboot system
4. Verify with `lms info --gpu`

---

## 6. Performance Monitoring

### AMD Adrenalin Metrics

Monitor GPU utilization during inference:

```
AMD Software → Performance → Metrics → GPU Compute
```

**Expected Behavior**:
- GPU Compute: 60-95% during active inference
- GPU VRAM: Matches model weight size + context buffer
- System RAM: Minimal usage during inference (weights in VRAM)

**Degraded Behavior**:
- GPU Compute: < 30% (possible DirectML fallback or CPU inference)
- System RAM: High usage during inference (VRAM overflow, system paging)

---

<div align="center">

*All technical decisions are documented here with authoritative references. Questions? Review the linked documentation.*

</div>
