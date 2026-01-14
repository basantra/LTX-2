# LTX-2 Technical Deep Dive

> **Reference**: [HuggingFace Paper Page](https://huggingface.co/papers/2601.03233)
> **Topic**: Architecture, Implementation, and Mathematical Foundations

---

## 1. Introduction: The "World Model" Approach

LTX-2 represents a shift from "Video Generation" to **"World Modeling"**. By modeling video and audio jointly, it learns the physics of the world (how objects move) and the acoustics (how they sound) simultaneously.

**Key Innovation**: Single-pass generation of $P(V, A | T)$ instead of the cascaded $P(V|T) \rightarrow P(A|V)$.

---

## 2. Mathematical Foundations

### Flow Matching
LTX-2 abandons standard DDPM (Denoising Diffusion Probabilistic Models) for **Flow Matching**.
- **Concept**: Instead of learning to remove noise, the model learns a **velocity field** $v_t(x)$ that transports the probability distribution from noise $p_1(x)$ to data $p_0(x)$ over time $[0, 1]$.
- **ODE Solver**: Inference is performing an Euler integration step along this vector field.
    $$ x_{t-dt} = x_t - v_\theta(x_t, t) \cdot dt $$

### RoPE (Rotary Positional Embeddings)
- **Video**: Uses **3D RoPE**.
    - Dimensions: $(Height, Width, Time)$.
    - Ensures the model understands spatial relationships *and* temporal causality.
- **Audio**: Uses **1D RoPE**.
    - Dimension: $(Time)$.
    - Audio is treated as a linear sequence of tokens.

---

## 3. Architecture Breakdown

### 3.1 The Input Protocol (`Modality` class)
Inputs are wrapped in a generic `Modality` dataclass (`modality.py`).
- separation of concerns: The generic transformer block doesn't "know" if it's processing video or audio pixels; it just processes sequences of `tokens` with associated `timesteps` and `embeddings`.

### 3.2 Transformer Block (`BasicAVTransformerBlock`)
The core processing unit. 
1. **AdaLN (Adaptive Layer Norm)**:
    - Regresses scale/shift parameters from the `timestep` embedding.
    - This "injects" the diffusion time into every layer.
2. **Cross-Modality Attention**:
    - **Crucial**: Audio tokens attend to Video tokens, and vice versa.
    - This is where synchronization happens.
3. **Self-Attention**:
    - Standard causal (or bidirectional, depending on mask) attention within each modality.
4. **Feed-Forward**:
    - Standard MLP.

### 3.3 The VAEs
- **Video VAE**: 
    - Compresses $128 \times H \times W$ video chunks into latent space.
    - Compression factors: $8 \times$ temporal, $32 \times$ spatial.
    - This high compression is necessary to fit 121 frames into memory.
- **Audio VAE**:
    - Compresses raw waveform into tokens.
    - Uses a smaller temporal compression factor ($4 \times$) to preserve high-frequency transients.

---

## 4. The Audio Bottleneck (Critical Analysis)

While the transformer is state-of-the-art, the **Audio VAE decoder** (Vocoder) uses an older **HiFi-GAN** architecture.
- **Limitation**: Operates at 24kHz.
- **Consequence**: Audio lacks "air" (>12kHz frequencies).
- **Opportunity**: Upgrading this component to **BigVGAN** (44.1kHz) is the highest-ROI improvement available (see `audio_vocoder_upgrade.md`).

---

## 5. Inference & Schedulers

### LTX2Scheduler
- Implementation: `schedulers.py`
- **SNR Shift**: As image resolution increases, the signal-to-noise ratio changes. Standard cosine schedules fail.
- **Solution**: The scheduler stretches the timestep distribution based on the total number of tokens (resolution * frames), ensuring consistent contrast and color saturation.

### Guidance
- **CFG (Classifier-Free Guidance)**:
    $$ \epsilon_{final} = \epsilon_{uncond} + s \cdot (\epsilon_{cond} - \epsilon_{uncond}) $$
- **STG (Spatio-Temporal Guidance)**:
    - LTX-2 implements advanced guidance by selectively perturbing specific attention heads (usually High-Frequency ones) to improve structure. (`perturbations` argument in `forward`).
