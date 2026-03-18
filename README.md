# Self-Hosted AI Agent

A self-hosted chat agent setup running on local hardware, using open source tooling.

## Hardware

| Component | Spec |
|-----------|------|
| CPU | AMD Ryzen 7 7800X3D (8 cores / 16 threads, up to 5 GHz) |
| RAM | 64 GB |
| GPU | NVIDIA GeForce RTX 4080 SUPER (16 GB VRAM) |
| Disk | 1.9 TB NVMe (~960 GB free) |
| OS | Arch Linux (kernel 6.19) |

**Model capacity:** 16 GB VRAM supports up to ~13B parameters at full precision, or 32B–70B models quantized (Q4/Q5) entirely on-GPU.

## Stack

- **[Ollama](https://ollama.com/)** — local model runtime, handles downloading and serving LLMs via a simple API
- **[Open WebUI](https://github.com/open-webui/open-webui)** — ChatGPT-like web interface that connects to Ollama

## Setup

### 1. Install Ollama

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

Start the Ollama service:

```bash
systemctl --user enable --now ollama
```

### 2. Pull a model

```bash
# Recommended: Qwen 2.5 32B (fits in 16 GB VRAM at Q4)
ollama pull qwen2.5:32b

# Alternative: Llama 3.3 70B quantized
ollama pull llama3.3:70b-instruct-q4_K_M
```

### 3. Install Open WebUI

Using Docker:

```bash
docker run -d \
  --name open-webui \
  --gpus all \
  -p 3000:8080 \
  -v open-webui:/app/backend/data \
  --add-host=host.docker.internal:host-gateway \
  -e OLLAMA_BASE_URL=http://host.docker.internal:11434 \
  --restart always \
  ghcr.io/open-webui/open-webui:cuda
```

Open WebUI is then available at [http://localhost:3000](http://localhost:3000).

## Models

| Model | VRAM | Notes |
|-------|------|-------|
| `qwen2.5:32b` | ~20 GB (Q4) | Strong general-purpose, code, reasoning |
| `llama3.3:70b-instruct-q4_K_M` | ~40 GB (Q4) | Needs offloading at 16 GB VRAM |
| `qwen2.5-coder:14b` | ~9 GB | Optimized for code |
| `mistral-small:22b` | ~13 GB | Fast, efficient |
