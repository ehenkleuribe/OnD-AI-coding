# DOCUMENTATION PLAN: RyzenAI 32GB Local Coding Setup

This plan details the modular documentation required for a high-performance RyzenAI local coding environment. The next engineer must follow this plan to build the final documentation suite.

## 1. Documentation Modules
- `README.md`: High-level entry point. Includes the architectural diagram and project objectives.
- `QUICKSTART.md`: Step-by-step CLI setup. Focuses on `winget` installation, tool pulling, and initial configuration.
- `SETUP.md`: Environment configuration. Focuses on Vulkan backend initialization and hardware constraints.
- `CONFIG.md`: `opencode.json` specification. Defines the enforcement of memory policies and context limits.
- `NOTES.md`: The authoritative technical justification. All design choices (Vulkan, memory policy) are anchored here with official links.

## 2. Technical Guidelines
- **Backend**: Vulkan must be the forced backend for all inference engines.
- **Memory Policy**: "Single-Active-Model" enforcement (only one model resident in memory).
- **Style**: Succinct, enthusiastic, clean aesthetic (consistent with the provided banner). Restrained emoticon use, avoid AI-cliché language.
- **CLI-First**: All setups must be performed via Windows Terminal (winget, CLI arguments).

## 3. Implementation Blueprint for Next Engineer

### 3.1. README.md (Visuals & Context)
- **Banner**: Include the project banner.
- **Diagram**: Use Mermaid.js for a clean visual of the "Build -> Architect" workflow.
- **Content**: Meta-objectives, project goal (optimizing 32GB RAM/RyzenAI), and navigation links to modular files.

### 3.2. QUICKSTART.md (Actionable Steps)
- **winget**: Installation commands for Ollama and LM Studio.
- **Initialization**: CLI commands to download Gemma 4B (Build) and Qwen 3.6 35B (Architect).
- **Verification**: Simple test-run command to confirm engine responsiveness.

### 3.3. SETUP.md (The "How-To")
- **Environment**: Define environment variables required to force the Vulkan backend (e.g., `OLLAMA_LLM_LIBRARY=vulkan`).
- **Memory Limits**: Instructions for configuring LM Studio/Ollama to strictly follow the 32k context / 1 concurrency limit.

### 3.4. CONFIG.md (opencode.json Specification)
- **Schema**: Provide the exact `opencode.json` structure.
- **Enforcement**: Document the key-value pairs required for the `single_resident` memory policy and context/concurrency constraints.

### 3.5. NOTES.md (The Authority)
- **Design Rationale**: Justification for Vulkan over DirectML (anchored in official llama.cpp/Ollama/LM Studio docs).
- **Constraints**: Documentation on why "Single-Active-Model" is required for stability on 32GB systems.
- **Authoritative Links**: All design decisions must be linked to official tool documentation.
