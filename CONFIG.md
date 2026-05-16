# CONFIG — opencode.json Specification

This document defines the `opencode.json` schema required to orchestrate the Build (Gemma 4B) and Architect (Qwen 3.6 35B) models with Vulkan acceleration on your RyzenAI 32GB system.

---

## 1. Full Configuration

Copy this into your project root as `opencode.json`:

```json
{
  "engine": {
    "concurrency": 1,
    "context_window": 32768,
    "single_resident": true,
    "unload_on_switch": true,
    "backend": "vulkan"
  },
  "models": {
    "build": {
      "id": "Gemma-4B-Build",
      "backend": "ollama",
      "description": "Rapid code iteration — lightweight, fast inference"
    },
    "architect": {
      "id": "qwen2.5-32b-instruct-q4_k_m",
      "backend": "lms",
      "description": "Deep reasoning & planning — parameter-rich architecture mode"
    }
  },
  "memory": {
    "max_resident_mb": 28000,
    "eviction_policy": "least_recently_used",
    "flush_on_switch": true
  },
  "logging": {
    "level": "info",
    "log_path": "./logs/opencode.log"
  }
}
```

---

## 2. Schema Reference

### `engine` — Inference Controls

| Key | Type | Default | Required | Description |
|:---|:---|:---|:---|:---|
| `concurrency` | int | `1` | Yes | Max parallel inference threads |
| `context_window` | int | `32768` | Yes | Token context length limit |
| `single_resident` | bool | `true` | Yes | Enforce one model in memory |
| `unload_on_switch` | bool | `true` | Yes | Auto-unload previous model |
| `backend` | string | `"vulkan"` | Yes | Inference backend (`vulkan` or `directml`) |

### `models` — Model Definitions

| Key | Type | Description |
|:---|:---|:---|
| `build.id` | string | Ollama model identifier |
| `build.backend` | string | Inference engine (`ollama`) |
| `architect.id` | string | LM Studio model identifier |
| `architect.backend` | string | Inference engine (`lms`) |

### `memory` — Memory Management

| Key | Type | Default | Description |
|:---|:---|:---|:---|
| `max_resident_mb` | int | `28000` | Maximum resident model size in MB |
| `eviction_policy` | string | `"least_recently_used"` | Model eviction strategy |
| `flush_on_switch` | bool | `true` | Force VRAM flush during model switch |

---

## 3. Usage

### Switch to Build Mode

```powershell
opencode --model build
```

### Switch to Architect Mode

```powershell
opencode --model architect
```

### Check Active Model

```powershell
opencode status
```

---

## 4. Memory Policy Enforcement

The `single_resident` policy operates as follows:

1. **Load Request Received**: `opencode` checks for any resident model
2. **Unload Previous**: If a model is loaded, it is evicted (`unload_on_switch`)
3. **VRAM Flush**: `flush_on_switch` forces GPU VRAM cleanup
4. **Load New Model**: New model loads into clean memory space
5. **Verify**: System memory and VRAM checked for stability

This prevents:
- OOM (Out of Memory) crashes
- System paging during inference
- Context corruption from stale weights

---

<div align="center">

*Need the technical rationale? Head to [NOTES.md](NOTES.md) for authoritative documentation.*

</div>
