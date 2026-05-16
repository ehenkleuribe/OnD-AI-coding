# SETUP — Environment & Vulkan Configuration

This document covers the environment configuration required to run Vulkan-accelerated inference on your RyzenAI iGPU with 32GB RAM.

---

## 1. Vulkan Backend Initialization

### 1.1 Environment Variables

Set the following system-level variables:

```powershell
# Force Vulkan backend for Ollama
[System.Environment]::SetEnvironmentVariable('OLLAMA_LLM_LIBRARY', 'vulkan', 'Machine')

# Enforce single active model (critical for 32GB systems)
[System.Environment]::SetEnvironmentVariable('OLLAMA_MAX_LOADED_MODELS', '1', 'Machine')

# Optional: Limit GPU layers for stability
[System.Environment]::SetEnvironmentVariable('OLLAMA_NUM_GPU', '99', 'Machine')
```

### 1.2 Vulkan ICD Verification

Ensure your AMD Vulkan Installable Client Driver (ICD) is properly registered:

```powershell
Get-ChildItem -Path "C:\Windows\System32\vk_icd*" | Select-Object Name
```

Expected output includes `vk_icd_amd64.dll` or similar AMD Vulkan ICD file.

If missing, install the latest **AMD Vulkan Runtime** from [AMD Drivers](https://www.amd.com/en/support).

---

## 2. Memory Management Constraints

### 2.1 Single-Active-Model Policy

On 32GB RAM systems, loading two models simultaneously causes system-level paging and inference degradation. The `opencode.json` enforces:

```json
{
  "engine": {
    "single_resident": true,
    "unload_on_switch": true
  }
}
```

This ensures:
- Only one model resident in VRAM/RAM at any time
- Automatic unloading of the previous model before the next load
- No system paging or OOM crashes during model switching

### 2.2 Context Window & Concurrency

| Parameter | Value | Rationale |
|:---|:---|:---|
| `context_window` | 32768 | Fits within 32GB envelope with model weights |
| `concurrency` | 1 | Prevents memory fragmentation during inference |

These values are enforced in `opencode.json` (see [`CONFIG.md`](CONFIG.md)).

---

## 3. LM Studio Configuration

### 3.1 Headless Daemon Setup

Start the LM Studio daemon from CLI:

```powershell
lms start --port 1234
```

### 3.2 Model Loading with Constraints

```powershell
lms load qwen2.5-32b-instruct-q4_k_m --context-length 32768 --parallel-requests 1
```

### 3.3 Model Unloading

```powershell
lms unload --all
```

---

## 4. Ollama Configuration

### 4.1 Create Custom Modelfile

Create `Build.Modelfile`:

```dockerfile
FROM gemma:4b
PARAMETER num_ctx 32768
PARAMETER num_gpu 99
PARAMETER num_thread 8
```

### 4.2 Build Custom Model

```powershell
ollama create Gemma-4B-Build -f Build.Modelfile
```

### 4.3 Run with Constraints

```powershell
ollama run Gemma-4B-Build
```

---

## 5. Verification Checklist

| Check | Command | Expected |
|:---|:---|:---|
| Vulkan library loaded | `ollama list` | No DirectML warnings |
| Single model loaded | `lms list` | One model showing `loaded` |
| GPU utilization | AMD Adrenalin → Performance | GPU Compute > 60% during inference |
| Memory pressure | Task Manager → Memory | No paging > 10% during model switch |

---

<div align="center">

*Ready for configuration? Head to [CONFIG.md](CONFIG.md) to set up opencode.*

</div>
