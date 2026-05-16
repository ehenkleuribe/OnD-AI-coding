# QUICKSTART — CLI Setup Guide

Get your RyzenAI 32GB system running with Vulkan-accelerated local models. All commands execute in **Windows Terminal**.

---

## 1. Install Tools

```powershell
# Install Ollama
winget install Ollama.Ollama --accept-package-agreements --accept-source-agreements

# Install LM Studio
winget install ElementLabs.LMStudio --accept-package-agreements --accept-source-agreements
```

Verify installation:

```powershell
ollama --version
lms --version
```

---

## 2. Configure Environment

Set Vulkan as the default inference backend:

```powershell
[System.Environment]::SetEnvironmentVariable('OLLAMA_LLM_LIBRARY', 'vulkan', 'Machine')
[System.Environment]::SetEnvironmentVariable('OLLAMA_MAX_LOADED_MODELS', '1', 'Machine')
```

Verify:

```powershell
$env:OLLAMA_LLM_LIBRARY
$env:OLLAMA_MAX_LOADED_MODELS
```

---

## 3. Pull Models

### Build Model — Gemma 4B

```powershell
ollama pull gemma:4b
```

### Architect Model — Qwen 3.6 35B

```powershell
lms get qwen2.5-32b-instruct-q4_k_m
```

> **Note**: The `q4_k_m` quantization balances context depth with memory footprint on 32GB systems.

---

## 4. Verify Engine Responsiveness

### Test Ollama

```powershell
ollama run gemma:4b "Hello, test message."
```

### Test LM Studio Daemon

```powershell
lms start
lms list
```

---

## 5. Configure opencode

Copy the `opencode.json` from [`CONFIG.md`](CONFIG.md) into your project root, then run:

```powershell
opencode --model build
opencode --model architect
```

---

## 6. Monitor GPU Utilization

Open **AMD Software: Adrenalin Edition** and navigate to:

```
Performance → Metrics → GPU Compute
```

Verify that model inference is driving GPU utilization above 60% during active prompts.

---

<div align="center">

*You're set. Head to [SETUP.md](SETUP.md) for environment deep-dive or [CONFIG.md](CONFIG.md) for opencode configuration.*

</div>
