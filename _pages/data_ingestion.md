---
title: 'Data Ingestion Stage for LLMs'
date: 2025-08-25
permalink: /datallm/
---

# Data Ingestion & Normalization → Tokenization → Metadata

A survey of how researchers and companies handle this stage in the LLM lifecycle.

---

## 1. Normalization
**Goal**: Standardize raw text into a consistent, model-ready format.

- **Common Techniques**  
  - Lowercasing (sometimes preserving case for languages where it matters).  
  - Unicode normalization (NFC/NFD).  
  - Removing boilerplate (ads, HTML tags, navigation).  
  - Handling punctuation, whitespace, accents, emojis.  
  - Sentence and paragraph segmentation.  

- **Approaches in Practice**  
  - **Web-scale pipelines** (C4, The Pile): boilerplate removal with tools like *jusText*, *trafilatura*.  
  - **Curated corpora** (Phi-3, Dolma): aggressive cleaning for textbook-style clarity.  
  - **Code datasets** (CodeParrot, StarCoder): normalize tabs/spaces, comments, and formatting.  
  - **Multilingual datasets** (XLM-R, mBERT): Unicode normalization, retain diacritics and scripts.  

- **Trade-offs**  
  - ✅ Cleaner data → better generalization.  
  - ❌ Over-cleaning can remove useful signals (capitalization, formatting).  

---

## 2. Tokenization
**Goal**: Convert text into tokens that the model understands.

- **Methods**  
  1. **Whitespace/word-based** (rare today)  
     - ✅ Intuitive  
     - ❌ Huge vocab, poor handling of rare words  
  2. **Character-level**  
     - ✅ Infinite coverage  
     - ❌ Long sequences → inefficient  
  3. **Subword-based** (BPE, WordPiece, UnigramLM)  
     - ✅ Balance between coverage & efficiency  
     - ❌ Can split awkwardly in morphologically rich languages  
  4. **Byte-level** (GPT-2 BPE, LLaMA, GPT-4)  
     - ✅ Universal UTF-8 coverage, no OOV  
     - ❌ Longer sequences for non-Latin scripts  
  5. **Morpheme-aware** (linguistic segmentation, limited adoption)  
     - ✅ Good for morphologically rich languages  
     - ❌ Hard to scale, resource-intensive  
  6. **Hybrid (char + subword)** (CANINE, ByT5, Charformer)  
     - ✅ Robust to typos, OCR noise  
     - ❌ Computationally heavier  

- **Trends**  
  - Subword remains dominant.  
  - Byte-level increasingly popular for robustness.  
  - Hybrids emerging for noisy/OCR-heavy settings.  

---

## 3. Metadata
**Goal**: Attach structured information to each text/document for filtering, weighting, and governance.

- **Examples of Metadata**  
  - Source domain, timestamp, language  
  - Quality score, deduplication hash  
  - Copyright/licensing info  
  - Safety labels (toxicity, violence, etc.)  

- **Approaches in Practice**  
  - **C4 (T5)**: domain + URL metadata for filtering.  
  - **Dolma (AI2)**: detailed metadata schema (source type, content type, quality).  
  - **Anthropic**: safety tags (violence, hate, toxicity categories).  
  - **OpenAI internal pipelines**: use metadata for data weighting (e.g., upweight Q&A, textbooks).  
  - **RAG systems (KILT, RA-DIT)**: fine-grained retrieval indices as metadata.  

- **Trade-offs**  
  - ✅ Enables flexible sampling, safety filters, reproducibility.  
  - ❌ Maintaining consistent schemas is costly; storage overhead.  

---

## Key Insights
- **Normalization**: Must balance cleanliness with preserving linguistic signals.  
- **Tokenization**: Subword is standard, byte-level is rising, hybrids for noise robustness.  
- **Metadata**: Moving toward richer, structured metadata for governance and weighted sampling.  
