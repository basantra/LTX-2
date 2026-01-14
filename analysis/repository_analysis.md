# LTX-2 Repository Analysis

> **Date**: January 14, 2026
> **Scope**: Structural and functional analysis of the LTX-2 monorepo

---

## 1. Repository Structure Overview

The LTX-2 project is organized as a **monorepo** managed with `uv`, containing three primary packages. This structure allows for tight integration while maintaining logical separation of concerns.

### Root Directory
- **`pyproject.toml`**: The workspace configuration file. It defines the build system (hatchling), workspace members (`packages/*`), and development dependencies (ruff, mypy, pytest).
- **`uv.lock`**: fast, cross-platform package manager lockfile, ensuring deterministic builds.
- **`LICENSE`**: Community License.

### Packages Breakdown

#### 📦 [`ltx-core`](/packages/ltx-core)
The heart of the project. Contains the model definitions and inference logic.
- **`src/ltx_core/model/`**:
    - **`transformer/`**: Implementation of the DiT (Diffusion Transformer) architecture.
        - `model.py`: Main `LTXModel` class.
        - `attention.py`: Attention mechanisms (potential modernization target).
        - `rope.py`: Rotary Positional Embeddings (3D for video, 1D for audio).
    - **`audio_vae/`**: Audio encoding/decoding logic (HiFi-GAN based vocoder).
    - **`video_vae/`**: Video compression model.
- **`src/ltx_core/components/`**: Schedulers (`LTX2Scheduler`) and helpers.

#### 🚀 [`ltx-pipelines`](/packages/ltx-pipelines)
Orchestration layer for running inference.
- **`src/ltx_pipelines/pipelines.py`**: Defines high-level pipelines like `TI2VidTwoStagesPipeline` (Text+Image to Video).
- Handles the glue logic between the Transformer, VAEs, and T5/Gemma encoders.

#### 🏋️ [`ltx-trainer`](/packages/ltx-trainer)
Training infrastructure.
- **`src/ltx_trainer/trainer.py`**: The training loop (accelerate-based).
- **`src/ltx_trainer/config.py`**: Pydantic-based configuration schemas.
- **`src/ltx_trainer/training_strategies/`**: Defines specific training tasks (e.g., `text_to_video`).

---

## 2. Key Architecture Decisions

### **Joint Audio-Video Generation**
Unlike competitors that generate video first and then generate audio (T2V → V2A), LTX-2 models $P(Video, Audio | Text)$ **jointly**.
- **Implementation**: `BasicAVTransformerBlock` in `ltx-core` process both modalities simultaneously with bidirectional cross-attention.
- **Benefit**: Superior synchronization (lip-sync).

### **Asymmetric Parameters**
- **Video**: ~14B parameters
- **Audio**: ~5B parameters
- **Rationale**: Video data is significantly more high-dimensional and complex than audio. Allocating capacity asymmetrically maximizes efficiency.

### **Flow Matching**
- Uses **Flow Matching** (an ODE-based generative framework) rather than standard DDPM.
- **Scheduler**: `LTX2Scheduler` adapts the step size based on token count, correcting for the "SNR shift" common in high-res generation.

---

## 3. Getting Started Guide

### Installation
The project uses `uv` for ultra-fast dependency management.

```bash
# Install uv
pip install uv

# Sync dependencies
uv sync --all-extras
```

### Running Inference
(Assuming weights are downloaded to `models/`)

```python
from ltx_pipelines import TI2VidTwoStagesPipeline
import torch

pipe = TI2VidTwoStagesPipeline.from_pretrained("Lightricks/LTX-2")
pipe.to("cuda")

video = pipe(
    prompt="A cinematic shot of a robot painting a canvas",
    negative_prompt="blur, distortion",
    width=768,
    height=512,
    num_frames=121
).video

# Save video...
```

---

## 4. Performance & Optimization

- **Flash Attention**: Supported in `attention.py` but requires `flash-attn` installed.
- **FP8 Support**: Present in `ltx-trainer` (via `bitsandbytes` or `TransformerEngine` hooks).
- **Gradient Checkpointing**: Essential for training the 19B model on anything less than an H100 cluster.

## 5. License & Usage
- **LTX-2 Community License**:
    - Free for non-commercial use, research, and companies with <$10M revenue.
    - Commercial license required for large enterprises.
