# Why HiFi-GAN Fails on Novel Patterns

> **Topic**: Analysis of Audio Artifacts in Non-Speech Generation

---

## The "Periodicity Bias" Problem

HiFi-GAN (the current vocoder in LTX-2) was engineered specifically for **TTS (Text-to-Speech)**. 
- **Mechanism**: It uses a **Multi-Period Discriminator (MPD)**.
- **Assumption**: The MPD assumes the input signal is composed of repeating waveforms (vocal cord vibrations) at various periods ($p=2, 3, 5, 7, 11...$).

### Failure Mode: Non-Periodic Sounds
When generating "Novel Patterns" that LTX-2 (a world model) encounters—such as:
- 🌊 Ocean waves
- 💥 Explosions
- 🏗️ Grinding metal
- 🥁 Complex drum transients

**HiFi-GAN fails because these are not periodic.** 
It attempts to force a periodic structure onto chaotic noise.
- **Result**: "Metallic" ringing artifacts, phase smearing, and a "robotic" texture overlaying organic sounds.

## The Fix: Snake Activations (BigVGAN)

Modern vocoders like BigVGAN use **Snake Functions**:
$$ f(x) = x + \frac{1}{\alpha} \sin^2(\alpha x) $$

- This learnable non-linearity allows the model to adapt to *any* frequency distribution, not just harmonic series.
- It can model chaotic, separate noise components much better than HiFi-GAN's fixed-period discriminators.

**Conclusion**: For a universal generator, HiFi-GAN is the wrong tool. It biases the "World Model" to sound like a "Speech Model".
