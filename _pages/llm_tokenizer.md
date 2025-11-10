---
title: 'Tokenizer Design: Choosing BPE, Unigram, and Vocabulary Size'
date: 2025-10-17
permalink: /llm_tokenizer/
---

The tokenizer is the bridge between human-readable text and model-interpretable tokens. Tokenizer choice affects efficiency, cross-lingual performance, generalization, and robustness to noise.

---

## 1. Goal
Transform raw text into tokens (subwords, characters, or bytes) in a way that:
- Efficiently represents all possible text (no out-of-vocabulary issues).
- Balances vocabulary size vs. sequence length.
- Supports multilingual text and special domains (code, math, emoji).
- Preserves semantic and morphological structure where helpful.

---

## 2. Why Tokenizer Design Matters
- Determines **context window efficiency**: smaller tokens → longer sequences; larger tokens → shorter but coarser inputs.
- Shapes **embedding layer size** (directly tied to vocab size).
- Affects **multilingual handling**, **robustness to noise**, and **downstream compression**.
- Influences **training cost per token** and **output granularity** in generation.

---

## 3. Major Tokenization Paradigms

| Type | Description | Examples | Pros | Cons |
|------|--------------|-----------|------|------|
| **Word-level** | Split by whitespace and punctuation. | Early NLP models (Word2Vec, GloVe). | Simple, interpretable. | Fails on rare words, inflected forms, or new words. |
| **Character-level** | Each character is a token. | CharRNNs, ByT5 variants. | No OOV issues; handles typos/OCR errors. | Long sequences → inefficient. |
| **Subword-based** | Split words into frequently occurring subunits. | BPE (GPT), WordPiece (BERT), Unigram (T5). | Compact, efficient, covers rare words. | Language bias (Latin-centric). |
| **Byte-level** | Operates directly on UTF-8 bytes. | GPT-2, LLaMA-2, GPT-4. | Universal coverage, no OOV. | Longer sequences for non-Latin scripts. |
| **Morpheme-aware** | Linguistically segmented tokens. | M-BERT extensions, PolyLM. | Excellent for morphologically rich languages. | Complex preprocessing; not universal. |
| **Character-subword hybrids** | Learn both granular and composed tokens. | CANINE, Charformer, ByT5. | Robust to spelling noise. | Slower training; complex encoding. |

---

## 4. Dominant Algorithms

### **1. Byte Pair Encoding (BPE)**
- Iteratively merges frequent symbol pairs into subwords.
- Used by: **GPT family, LLaMA-1/2**, **Falcon**, **Mixtral**.
- ✅ *Pros:* Efficient, easy to implement, compact.  
- ❌ *Cons:* Deterministic merges can overfit to training corpus frequency; hard to adapt multilingual text.

### **2. WordPiece**
- Optimizes token likelihood using a unigram language model over subwords.
- Used by: **BERT**, **RoBERTa**, **ALBERT**.
- ✅ *Pros:* Probabilistic → smoother subword splits.  
- ❌ *Cons:* Slightly slower training; vocabulary harder to interpret.

### **3. Unigram Language Model (SentencePiece)**
- Maintains a set of candidate subwords, prunes low-likelihood ones.
- Used by: **T5**, **Flan-T5**, **Mistral**, **Gemma**, **Phi-3**.
- ✅ *Pros:* Better coverage; probabilistic merges; multilingual friendly.  
- ❌ *Cons:* More complex training; not easily reversible.

### **4. Byte-Level BPE**
- BPE applied directly to raw bytes (0–255).
- Used by: **GPT-2/3/4**, **LLaMA-2**, **Gemini**, **Claude 3**.
- ✅ *Pros:* Universal UTF-8 support, no special casing for languages.  
- ❌ *Cons:* Creates more tokens for non-Latin text (e.g., Chinese, Arabic).

### **5. SentencePiece (Framework)**
- Framework supporting Unigram or BPE with built-in normalization and pre/post-processing.
- Used by: **T5**, **LLaMA**, **Gemma**, **Phi-3**.
- ✅ *Pros:* Language-independent, reproducible, standalone.  
- ❌ *Cons:* Requires extra post-processing for detokenization fidelity.

---

## 5. Vocabulary Size Selection

| Model | Vocab Size | Rationale |
|--------|-------------|------------|
| **GPT-3 (OpenAI)** | 50,257 | Byte-BPE optimized for English + symbols. |
| **LLaMA-3 (Meta)** | 128,000 | Expanded multilingual coverage. |
| **T5 (Google)** | 32,000 | Tradeoff between quality and efficiency. |
| **Mistral (MistralAI)** | 32,000 | Same as LLaMA SentencePiece vocabulary. |
| **Phi-3 (Microsoft)** | ~64,000 | Balanced for code + text + reasoning. |
| **Gemini (Google)** | 100,000+ | Multi-modal and multilingual tokens. |

**Key Trade-offs:**
- **Smaller vocab** → longer sequences, cheaper embeddings.  
- **Larger vocab** → shorter sequences, higher embedding/memory cost.  
- Rule of thumb: Vocabulary ≈ √(dataset tokens) is efficient for web-scale LLMs.

---

## 6. Normalization & Preprocessing in Tokenization
Tokenizer design also includes **text normalization** before token splitting.

- Unicode normalization (NFC/NFD).  
- Lowercasing (if language-safe).  
- Removing control characters.  
- Unicode escape handling (`\uXXXX`).  
- Whitespace collapsing.  
- Script detection for multilingual tokenizers.

**Examples:**
- *SentencePiece* handles normalization internally.  
- *GPT-4 tokenizer* normalizes punctuation and accents for consistency.  

---

## 7. Multilingual and Domain-Specific Tokenization

### **Multilingual**
- Mix multiple languages into the same tokenizer training corpus.  
- Reduce bias toward high-frequency Latin scripts.  
- Use **Unigram** or **Byte-BPE** for open-vocabulary support.
- **Examples:**  
  - *XLM-R* uses SentencePiece Unigram across 100 languages.  
  - *LLaMA-3* introduces balanced script coverage and rebalanced merges.  

### **Code and Math Domains**
- Tokenize identifiers, operators, and whitespace meaningfully.  
- May use specialized subword vocabularies for programming symbols.  
- **Examples:**  
  - *CodeParrot*, *StarCoder*, *CodeLLaMA* use domain-specific BPE with token-level code comments.  
  - *DeepSeek-Coder* integrates AST-level tokenization for structural awareness.

---

## 8. Evaluation Criteria for Tokenizers

| Criterion | Metric / Approach |
|------------|------------------|
| **Coverage** | % of text represented without fallback or rare tokens |
| **Efficiency** | Avg. tokens per word or per character |
| **Compression** | Total tokens needed per dataset |
| **Reversibility** | Can detokenize exactly to original text? |
| **Cross-lingual balance** | Equal token efficiency across scripts |
| **Robustness to noise** | Tolerance to typos, OCR errors, emoji, code-mixing |

**Benchmarks:** Token-per-word ratios, downstream loss differences, multilingual perplexity comparisons.

---

## 9. Emerging Tokenization Directions

1. **Tokenizer-Free Models**
   - Character or byte embedding models (e.g., *ByT5*, *CANINE*, *Charformer*).
   - Goal: eliminate pre-tokenization bias.
   - Tradeoff: higher sequence length, more compute.

2. **Learned Tokenizers / Adapters**
   - Tokenizers co-trained with the model.
   - Examples: *Megabyte* (DeepMind, 2024) trains discrete compression layers end-to-end.

3. **Dynamic Vocabularies**
   - Vocab evolves as new data appears (domain adaptation, code updates).
   - Requires retraining embeddings or using adapters.

4. **Semantic Tokenization**
   - Future idea: tokenize based on meaning units or dependency trees rather than frequency.
   - Promising for reasoning and dialogue tasks.

---

## 10. Key Takeaways
- **BPE and Unigram remain the industry standard**, with Byte-BPE dominating new general-purpose LLMs.  
- **Vocabulary size** is a performance-efficiency tradeoff; 30k–120k tokens is the current sweet spot.  
- **Multilinguality and domain adaptation** drive new tokenizer innovations.  
- **Emerging trend:** moving toward **byte-level or tokenizer-free models** to reduce human preprocessing bias.

