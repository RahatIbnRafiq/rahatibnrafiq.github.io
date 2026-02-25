
---
title: 'LLM Architectures'
date: 2026-2-25
permalink: /llm_architecture/
---


# Architecture Selection for LLMs:
## Transformers, Mixture-of-Experts (MoE), and Hybrid Models

Architecture determines scaling behavior, compute efficiency, context length capability, and reasoning capacity.
Modern LLM research is largely an exploration of architectural efficiency under scaling constraints.

---

# 1. Transformer Architecture (Dense Models)

## 1.1 Overview

The Transformer (Vaswani et al., 2017) is the dominant architecture for LLMs.
Decoder-only Transformers are now standard for autoregressive language modeling.

Core Components:
- Multi-head self-attention
- Feed-forward MLP blocks
- Residual connections
- Layer normalization
- Positional encoding (RoPE, ALiBi, etc.)

Used by:
- GPT family (OpenAI)
- LLaMA (Meta)
- Claude (Anthropic)
- Gemini (Google DeepMind)
- Mistral
- Falcon

---

## 1.2 Why Transformers Work

- Self-attention enables global token interactions.
- Scales predictably with parameters and data.
- Exhibits strong scaling laws.
- Simple, parallelizable during training.

---

## 1.3 Strengths

✔ Predictable scaling behavior  
✔ Excellent reasoning and few-shot ability  
✔ Mature ecosystem and tooling  
✔ Stable distributed training  

---

## 1.4 Weaknesses

✘ Quadratic attention cost O(n²)  
✘ Memory heavy at long context  
✘ Dense compute — all parameters activated every token  
✘ Expensive inference for very large models  

---

## 1.5 Variants of Transformer Improvements

### A. Efficient Attention Variants
- FlashAttention
- Linear attention (Performer, Linformer)
- Sliding window attention (Longformer, Mistral)
- Sparse attention (BigBird)

### B. Positional Encoding Improvements
- RoPE (used in LLaMA, GPT-4)
- ALiBi
- LongRoPE scaling
- NTK-aware scaling

### C. Architectural Tweaks
- SwiGLU activations (LLaMA)
- RMSNorm instead of LayerNorm
- Parallel attention & MLP blocks
- Multi-query attention (MQA)

Trend:
Transformers remain dominant, but highly optimized.

---

# 2. Mixture-of-Experts (MoE)

## 2.1 Overview

MoE introduces conditional computation:
Instead of activating all parameters per token,
a routing network selects a small subset of expert layers.

Key Idea:
Sparse activation → larger parameter count at same FLOPs.

Used by:
- Mixtral (Mistral AI)
- DeepSeek-MoE
- GPT-4 (rumored MoE-style)
- Google Switch Transformer
- GLaM

---

## 2.2 Architecture Concept

Standard Transformer block:
    Attention → MLP

MoE block:
    Attention → Router → Top-k Experts (MLPs)

Only k experts (e.g., 2 out of 8) are activated per token.

---

## 2.3 Advantages

✔ Massive parameter count without proportional inference cost  
✔ Better quality-per-FLOP  
✔ Specialized experts (code, math, multilingual, etc.)  
✔ Efficient scaling beyond dense limits  

Example:
Mixtral 8x7B → behaves like ~45B dense model but with lower inference cost.

---

## 2.4 Challenges

✘ Load balancing between experts  
✘ Routing instability  
✘ Harder distributed training  
✘ Memory fragmentation  
✘ Complex deployment  

---

## 2.5 Research Variants

- Switch Transformer (Top-1 routing)
- Top-2 routing (more stable)
- Hierarchical MoE
- Expert choice routing
- Mixture-of-depth (conditional layers)

Trend:
MoE increasingly used in frontier-scale LLMs.

---

# 3. Hybrid Architectures

Hybrid models combine Transformers with other sequence models to overcome quadratic attention and scaling limits.

---

## 3.1 State-Space Models (SSMs)

Examples:
- Mamba
- Mamba-2
- S4
- Hyena

Key idea:
Replace attention with linear-time state-space recurrence.

Advantages:
✔ Linear scaling O(n)
✔ Very long context (100k+ tokens)
✔ Lower memory footprint

Disadvantages:
✘ Less mature ecosystem
✘ Historically weaker reasoning than attention
✘ Harder interpretability

Recent Trend:
Mamba-2 shows competitive performance with Transformers at lower cost.

---

## 3.2 Transformer + SSM Hybrid

Combine attention layers with state-space layers.

Examples:
- Jamba (AI21)
- Hybrid-Mamba models
- RetNet

Goal:
Keep attention for reasoning,
use SSM for long-range memory.

---

## 3.3 Retrieval-Augmented Architectures

External memory integration:
- RAG
- RETRO (DeepMind)
- KNN-LM
- Memorizing Transformers

Idea:
Model consults external datastore instead of internalizing all knowledge.

Advantages:
✔ Smaller core model
✔ Better factual accuracy
✔ Update knowledge without retraining

Disadvantages:
✘ Requires retrieval infra
✘ Latency overhead
✘ Retrieval errors propagate

---

## 3.4 Multimodal Hybrid Models

Combine:
- Vision Transformers (ViT)
- Audio encoders
- Text Transformers

Examples:
- GPT-4o
- Gemini
- Claude 3
- LLaVA
- Kosmos

Architecture includes modality encoders + shared transformer core.

---

# 4. Dense vs MoE vs Hybrid: Comparison

| Architecture | Compute | Parameter Scaling | Long Context | Complexity | Current Adoption |
|--------------|----------|------------------|--------------|------------|------------------|
| Dense Transformer | High | Linear with cost | Limited (quadratic) | Low | Very High |
| MoE | Moderate per token | Very High | Same as Transformer | High | Growing |
| State-Space | Low per token | Moderate | Excellent | Medium | Emerging |
| Hybrid (Transformer + SSM) | Balanced | High | Excellent | High | Emerging |
| Retrieval-Augmented | Lower core model | External memory | Unlimited via DB | High infra | Growing |

---

# 5. Industry Trends (2024–2026)

1. Dense Transformers still dominate <70B scale.
2. MoE is favored for frontier-scale (GPT-4-class).
3. Hybrid SSM models gaining traction for long-context efficiency.
4. Retrieval + small core model gaining popularity for enterprise use.
5. Compute efficiency is now more important than raw parameter count.

---

# 6. Key Takeaways

- Transformers remain the baseline.
- MoE improves quality-per-compute.
- Hybrid models aim to solve long-context and efficiency bottlenecks.
- Architecture innovation now focuses on compute efficiency, not just scaling size.
- Frontier labs treat routing and scaling strategies as competitive secrets.
