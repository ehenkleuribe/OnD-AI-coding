<div align="center">

# RyzenAI Local Coding Setup

### 32GB RAM · Ryzen iGPU · Vulkan Backend

<div style="margin: 20px 0;">
  <img src="https://img.shields.io/badge/Backend-Vulkan-orange?style=for-the-badge&logo=vulkan" alt="Vulkan Backend" />
  <img src="https://img.shields.io/badge/OS-Windows%2011%2024H2-blue?style=for-the-badge&logo=windows" alt="Windows 11" />
  <img src="https://img.shields.io/badge/Engine-opencode-purple?style=for-the-badge" alt="opencode" />
</div>

**CLI-first local LLM orchestration for RyzenAI systems.**  
Configure your 32GB RAM rig to run Gemma 4B (Build) and Qwen 3.6 35B (Architect) with Vulkan acceleration.

</div>

---

## 🎯 Objectives

| Goal | Detail |
|:---|:---|
| **Performance-First** | Vulkan backend for iGPU acceleration on Ryzen AI hardware |
| **Stability** | Single-Active-Model memory policy prevents system paging |
| **CLI-Driven** | All setup via Windows Terminal — no GUI required |
| **Reproducible** | `winget` installs, documented configs, authoritative references |

---

## 🏗 Architecture Overview

```mermaid
graph TD
    A[User] -->|opencode CLI| B[opencode.json]
    B -->|Build Mode| C[Gemma 4B · Vulkan]
    B -->|Architect Mode| D[Qwen 3.6 35B · Vulkan]
    C -->|unload| E[Memory Manager]
    D -->|unload| E
    E -->|load| C
    E -->|load| D
    C -->|DirectML/Vulkan| F[Ryzen iGPU]
    D -->|DirectML/Vulkan| F
    style C fill:#4A90E2,color:#fff
    style D fill:#FF6B35,color:#fff
    style E fill:#2C3E50,color:#fff
    style F fill:#E74C3C,color:#fff
```

**Key Design Decisions:**
- **Single Active Model**: Only one model resident in memory at a time
- **32k Context / 1 Concurrency**: Optimized for 32GB RAM stability
- **Vulkan Backend**: Hardware-accelerated inference via Vulkan ICD

---

## 📁 Documentation Structure

| File | Purpose |
|:---|:---|
| [`QUICKSTART.md`](QUICKSTART.md) | Step-by-step CLI setup guide |
| [`SETUP.md`](SETUP.md) | Environment configuration & Vulkan setup |
| [`CONFIG.md`](CONFIG.md) | `opencode.json` schema & usage |
| [`NOTES.md`](NOTES.md) | Technical justification & authoritative references |

---

## ⚡ Quick Links

- [Start Setup](QUICKSTART.md) — Get running in 5 minutes
- [Configure opencode](CONFIG.md) — Memory policy & model definitions
- [Technical Notes](NOTES.md) — Why we chose Vulkan, memory limits, and constraints

---

<div align="center">

*Built for the RyzenAI community. Optimize. Iterate. Deploy.*

</div>
