# local ai setup

A self-hosted chat agent setup running on local hardware, using open source tooling.

## Hardware

### Desktop (primary)

| Component | Spec |
|-----------|------|
| CPU | AMD Ryzen 7 7800X3D (8 cores / 16 threads, up to 5 GHz) |
| RAM | 64 GB |
| GPU | NVIDIA GeForce RTX 4080 SUPER (16 GB VRAM) |
| Disk | 1.9 TB NVMe |
| OS | Arch Linux (kernel 6.19) |

**Model capacity:** 16 GB VRAM supports up to ~13B parameters at full precision, or 32B–70B models quantized (Q4/Q5) entirely on-GPU.

### MacBook Air M4 (to be evaluated)

| Component | Spec |
|-----------|------|
| CPU | Apple M4 (10-core) |
| RAM | 32 GB (unified memory) |
| GPU | Apple M4 integrated (unified memory) |
| Disk | TBD |
| OS | macOS (TBD) |

**Model capacity:** 32 GB unified memory supports up to ~26B parameters at full precision, or larger models quantized.

## Stack

- **[Ollama](https://ollama.com/)** — local model runtime, handles downloading and serving LLMs via a simple API
- **[Open WebUI](https://github.com/open-webui/open-webui)** — ChatGPT-like web interface that connects to Ollama
- **[Aider](https://aider.chat/)** — terminal-based AI coding agent, edits files and commits changes

## Setup

### Desktop (Arch Linux)

#### 1. Install Ollama

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

Start the Ollama service (installs as a system service, not user):

```bash
sudo systemctl enable --now ollama
```

Configure Ollama to listen on all interfaces (required for Docker to reach it). Create a systemd override:

```bash
sudo mkdir -p /etc/systemd/system/ollama.service.d
sudo nvim /etc/systemd/system/ollama.service.d/override.conf
```

Add:

```ini
[Service]
Environment="OLLAMA_HOST=0.0.0.0"
```

Reload and restart:

```bash
sudo systemctl daemon-reload && sudo systemctl restart ollama
```

Verify it's running and listening on all interfaces:

```bash
systemctl is-active ollama
ss -tlnp | grep 11434  # should show 0.0.0.0:11434, not 127.0.0.1
```

#### 2. Pull a model

```bash
ollama pull qwen2.5-coder:14b
```

#### 3. Install NVIDIA Container Toolkit

Required for Docker to access the GPU:

```bash
sudo pacman -S nvidia-container-toolkit
```

Configure Docker to use the NVIDIA runtime:

```bash
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker
```

Generate the CDI spec so Docker can discover the correct NVIDIA library versions:

```bash
sudo nvidia-ctk cdi generate --output=/etc/cdi/nvidia.yaml
```

> **Note:** Without the CDI spec regeneration, Docker may reference stale library versions and fail to start the container.

#### 4. Start Open WebUI

Using Docker Compose (see `docker-compose.yml`):

```bash
docker compose up -d
```

Open WebUI is then available at [http://localhost:3000](http://localhost:3000).

#### 5. Install Aider

Install `uv` first:

```bash
sudo pacman -S uv
```

Install Aider using Python 3.12 (required — scipy has no pre-built wheel for Python 3.14):

```bash
uv tool install aider-chat --python 3.12
```

Set the Ollama API base URL permanently (fish shell):

```bash
set -Ux OLLAMA_API_BASE http://localhost:11434
```

Run Aider in any project directory:

```bash
OLLAMA_API_BASE=http://localhost:11434 aider --model ollama/qwen2.5-coder:14b
```

> **Tested models:**
>
> - `qwen2.5-coder:14b` — recommended, fits fully in VRAM (~9 GB), optimized for code, fast response times
> - `qwen2.5:32b` — tested but too slow for interactive use; partially offloads to CPU RAM at 16 GB VRAM

### MacBook Air M4 (to be evaluated)

> Setup and model evaluation to be documented.

## Models

### Desktop

| Model | VRAM | Notes |
|-------|------|-------|
| `qwen2.5-coder:14b` | ~9 GB | Optimized for code — recommended for Aider |
| `qwen2.5:32b` | ~20 GB (Q4) | Strong general-purpose; too slow for Aider at 16 GB VRAM |
| `llama3.3:70b-instruct-q4_K_M` | ~40 GB (Q4) | Needs offloading at 16 GB VRAM |
| `mistral-small:22b` | ~13 GB | Fast, efficient |

### MacBook Air M4

> To be evaluated.
