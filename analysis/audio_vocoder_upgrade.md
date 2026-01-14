# Audio Vocoder Upgrade Recommendation

> **Component**: Audio VAE Decoder
> **Problem**: Limited High-Frequency Fidelity (24kHz cap)

---

## 1. The Bottleneck: HiFi-GAN (24kHz)

LTX-2 currently uses a **HiFi-GAN** vocoder operating at **24kHz**.
- **Architecture**: `ltx_core/model/audio_vae/vocoder.py`
- **Nyquist Limit**: 12kHz. Frequencies above 12kHz are completely lost.
- **Perceptual Result**: Audio sounds muffled, lacking "air" and brilliance. Good for speech, bad for music/SFX.
- **Inductive Bias**: HiFi-GAN optimizes for *periodicity* (speech), causing artifacts on *aperiodic* textures (rain, explosions).

---

## 2. The Solution: Modern Vocoders

We recommend swapping the vocoder (Decoder only). This **does NOT** require retraining the massive transformer.

### Option A: BigVGAN (Quality King) 👑
- **Why**: Uses **Snake Activations** ($x + sin^2x$).
- **Benefit**: Generalized perfectly to non-speech sounds (music, noise).
- **Upgrade**: Train a BigVGAN model to map LTX-2's *existing* latent Mels to **44.1kHz** waveforms (Super-Resolution).
- **Result**: "DVD Quality" audio.

### Option B: Vocos (Speed King) ⚡
- **Why**: Uses iSTFT (Fourier Transform) instead of deep convolutions.
- **Benefit**: **13x Faster Inference**.
- **Use Case**: Real-time applications.

---

## 3. Implementation Plan

**Goal**: Release `audio_decoder_pro.safetensors`.

1.  **Dataset Preparation**:
    - Take a high-quality dataset (e.g., AudioSet, 44.1kHz).
    - Encode it using LTX-2's *frozen* Audio Encoder to get Latents.
2.  **Training**:
    - Train **BigVGAN-v2** to map these Latents -> 44.1kHz Audio.
    - Loss: Multi-Scale Spectral Loss + GAN Loss.
3.  **Deployment**:
    - Drop-in replacement for `vocoder.py`.
    - No changes to the core Transformer needed.

**ROI**: High. Significant quality boost for <1% of the original training compute.
