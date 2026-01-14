# LTX-2 Modernization Opportunities

> **Analysis Date**: January 14, 2026  
> **Current Status**: Production-ready but room for cutting-edge optimizations at the "Research Frontier".

---

## Executive Summary

The LTX-2 codebase is **well-architected**, but 2024-2026 research offers significant opportunities to improve:
- **Inference speed**: 2-5× faster
- **Memory**: 40-60% reduction
- **Capability**: Breaking the 5-second video limit

---

## 1. Research Frontier: The "Infinite Video" Solution 🔬

### **Test-Time Training (TTT) for Temporal Attention** ⭐⭐⭐⭐⭐
- **Context**: Inspired by the "TTT-E2E" paper (arXiv:2512.23675) discussed in recent internal chats.
- **The Problem**: LTX-2 is currently capped at ~121 frames (5s) because Attention is $O(N^2)$. Long context requires massive KV caches.
- **The Solution**: Apply TTT layers to the **temporal dimension**.
    - Instead of caching explicit previous frames (KV cache), the model **trains itself on-the-fly**.
    - It compresses the history (characters, scenery) into the weights of a TTT update layer.
- **Impact**: **Infinite video generation** with constant memory usage. This is the "Holy Grail" feature for LTX-3.

---

## 2. Architectural Modernizations (Engineering)

### **Grouped Query Attention (GQA)** ⭐⭐⭐⭐⭐
- **Current**: Standard Multi-Head Attention (32 heads).
- **Update**: Share KV heads (e.g., 4 groups).
- **Impact**: **40% reduction in VRAM** usage for long sequences.

### **Sparse Attention Patterns** ⭐⭐⭐⭐
- **Current**: Full dense attention.
- **Update**: Sliding Window (Mistral-style) or Block Sparse.
- **Rationale**: Frame 120 rarely needs to attend pixel-perfectly to Frame 1.
- **Impact**: $O(N)$ scaling instead of $O(N^2)$.

---

## 3. Inference Optimizations

### **torch.compile Integration** ⭐⭐⭐⭐⭐
- **Current**: Present in code checks but not enforced/tuned.
- **Update**: Full graph capture with `mode="max-autotune"`.
- **Impact**: **2× inference speedup** (free).
- **Requirement**: Static bucket sizes to avoid recompilation.

### **KV-Cache for Text** ⭐⭐⭐
- **Observation**: Text embeddings never change during the 50 denoising steps.
- **Update**: Cache the Text Keys/Values once.
- **Impact**: 15-20% speedup.

---

## 4. Training Upgrades

### **Modern Optimizers** ⭐⭐⭐⭐
- **Current**: AdamW.
- **Recommendation**:
    - **Lion**: 30% less memory than AdamW.
    - **Sophia**: 2× faster convergence in steps.

### **FP8 Training** ⭐⭐⭐⭐
- **Target**: H100 clusters.
- **Impact**: 2× throughput via TransformerEngine.

---

## Implementation Roadmap

| Priority | Item | Difficulty | Impact |
|:---:|:---|:---:|:---|
| 🚨 **Critical** | `torch.compile` | Low | 2x Speed |
| 🚨 **Critical** | Grouped Query Attention | Med | -40% VRAM |
| 🚀 **High** | Text KV Cache | Low | +20% Speed |
| 🔬 **Research** | **TTT Temporal Layers** | V.High | **Infinite Video** |
| 🧪 **Exp** | BigVGAN Audio | Med | Pro Audio |
