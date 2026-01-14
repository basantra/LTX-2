# LTX-2 Modernization Backlog

> **Status**: Active
> **Source**: [Modernization Opportunities](./modernization_opportunities.md)
> **Goal**: Convert strategic opportunities into actionable engineering & research tasks.

---

## 🏗️ Epic 1: Performance Engineering (Immediate Impact)
**Goal**: Maximize inference / training speed and minimize memory footprint.

### [P0] Enable `torch.compile`
- [ ] **Task**: Wrap `LTXModel` forward pass with `torch.compile(mode="max-autotune")`.
- [ ] **Task**: Define static input bucket sizes to prevent recompilation.
- [ ] **Acceptance**: >1.8x inference speedup verified on benchmark.
- **Complexity**: Low (SP: 3)

### [P0] Implement Grouped Query Attention (GQA)
- [ ] **Task**: Refactor `Attention` class in `attention.py` to support `kv_groups` parameter.
- [ ] **Task**: Implement GQA projection logic (share K/V heads).
- [ ] **Task**: Create weight conversion script for 14B model (if applicable) or prepare training config.
- **Complexity**: Medium (SP: 5)

### [P1] Cache Text Embeddings
- [ ] **Task**: Modify `Attention` forward pass to accept cached key/values.
- [ ] **Task**: Update Inference Pipeline to compute text embeds once and pass to loop.
- **Complexity**: Low (SP: 2)

---

## 🔬 Epic 2: Audio Fidelity (Quality)
**Goal**: Solve the 12kHz frequency cutoff.

### [P1] BigVGAN Integration
- [ ] **Task**: Set up training pipeline to extract Latent Encodings from high-res (44.1kHz) audio dataset.
- [ ] **Task**: Train BigVGAN-v2 generator to map `Latents -> 44.1kHz Waveform`.
- [ ] **Task**: Integrate new `Vocoder` class into `audio_vae/`.
- **Complexity**: Medium (SP: 8)

---

## 🧪 Epic 3: Research Frontier (Infinite Video)
**Goal**: Break the 5-second barrier using Test-Time Training.

### [P2] TTT Temporal Layer Prototype
- [ ] **Research**: Literature review of "TTT-E2E" (arXiv:2512.23675) implementation details.
- [ ] **Task**: Fork `BasicAVTransformerBlock` to create `TTTTransformerBlock`.
- [ ] **Task**: Replace temporal self-attention with a TTT update layer (Mamba/RNN style).
- [ ] **PoC**: Train a small-scale model (100M params) on long video sequences to validate memory scaling.
- **Complexity**: Very High (SP: 21)

---

## 🛠️ Epic 4: Infrastructure & Training
**Goal**: Scale training to H100s and massive datasets.

### [P1] Modern Optimizers
- [ ] **Task**: Add `Lion` and `Sophia` to `ltx-trainer/optimizer.py`.
- [ ] **Task**: Benchmark convergence speed vs. AdamW on 1B subset.
- **Complexity**: Low (SP: 3)

### [P2] WebDataset Support
- [ ] **Task**: Implement `wds` data loader in `ltx-trainer/data.py`.
- [ ] **Task**: Support S3 streaming for training data.
- **Complexity**: Medium (SP: 5)
