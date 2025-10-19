---
title: 'Data Filtering Stage for LLMs'
date: 2025-10-17
permalink: /llm_data_filtering/
---

# Filtering: Deduplication, Low-Quality, and Unsafe Content Removal

This phase ensures that training data is **clean, diverse, and safe**, removing redundancies, low-value text, and harmful material before model training.

---

## 1. Deduplication
**Goal:** Prevent repeated text from dominating training and skewing gradients or memorization.

### Approaches
1. **Exact Deduplication**
   - **Hash-based**: Compute hashes (e.g., SHA1, MD5, MinHash) for entire documents or text chunks and remove duplicates.
   - **Example:** *The Pile* and *Dolma* used hash-based filtering across sources.
   - ✅ *Advantages:* Simple, fast, scales well to billions of documents.  
   - ❌ *Disadvantages:* Misses near-duplicates with minor edits or whitespace differences.

2. **Near-Duplicate Detection**
   - **MinHash / SimHash / Locality-Sensitive Hashing (LSH):** Detects texts with high similarity based on token shingles (n-grams).
   - **Embedding-based deduplication:** Use sentence or document embeddings (e.g., Sentence-BERT, MiniLM) and cluster by cosine similarity.
   - **Example:** *C4 (T5)* and *Pile v2* use LSH-based near-duplicate removal; *OpenAI* and *Anthropic* use embedding-based clustering for fine-grained deduplication.
   - ✅ *Advantages:* Removes paraphrased or lightly modified duplicates.  
   - ❌ *Disadvantages:* Expensive for large datasets; risk of removing legitimate paraphrases.

3. **Cross-Source Deduplication**
   - Detects overlap across datasets (e.g., same Wikipedia articles in multiple corpora).
   - **Example:** *Dolma* and *RefinedWeb* maintain global deduplication hashes across sources.

---

## 2. Low-Quality Content Removal
**Goal:** Remove noise, boilerplate, or irrelevant content (e.g., link lists, spam, logs).

### Approaches
1. **Heuristic Filtering**
   - Use rules like:  
     - Minimum/maximum length thresholds.  
     - Ratio of punctuation, digits, or stopwords.  
     - Boilerplate detection (HTML menus, templates).  
     - Language ID confidence threshold.  
   - **Tools:** *jusText*, *trafilatura*, *boilerpy3*, *ftfy*, *langdetect*.
   - ✅ *Advantages:* Simple and interpretable.  
   - ❌ *Disadvantages:* Hard to tune across diverse domains; can remove niche useful text.

2. **Model-Based Quality Scoring**
   - Train small classifiers or LLMs to score text on fluency, coherence, or informativeness.
   - **Example:** *Dolma*, *RefinedWeb*, and *FineWeb* assign “quality scores” using smaller LMs or GPT-like evaluators.
   - ✅ *Advantages:* Adapts to nuanced quality criteria.  
   - ❌ *Disadvantages:* Adds subjective bias; labeling or training such scorers is costly.

3. **Readability & “Textbookness” Filters**
   - Rank or filter based on readability metrics (Flesch–Kincaid), or “didactic” clarity (as in *Phi-3* data curation).
   - ✅ *Advantages:* Boosts reasoning and factual training signals.  
   - ❌ *Disadvantages:* Reduces domain diversity (creative, informal text).

4. **Token Distribution Checks**
   - Detect statistically unusual token distributions that indicate malformed or adversarial content.
   - **Example:** Used in *OpenWebMath* and *RedPajama* to remove noise.

---

## 3. Unsafe Content Filtering
**Goal:** Remove or tag text that violates safety, privacy, or compliance policies.

### Categories
- **PII (Personally Identifiable Information):**
  - Regex & ML-based detection (emails, addresses, SSNs, phone numbers).  
  - **Example:** *Anthropic* and *OpenAI* scrub all PII via regex + entity models.

- **Hate, Harassment, Violence, Sexual Content, Self-harm:**
  - Use rule-based keyword lists, toxicity classifiers (Perspective API, Detoxify), or LLM-based moderation models.
  - **Example:** *Anthropic’s Constitutional AI* uses model-assisted labeling for safety.  
  - **Meta’s LLaMA-3 pipeline** integrates toxicity classifiers before pretraining.

- **Copyright & Legal Filtering:**
  - Detect copyrighted material via URL filtering, license detection, or using datasets with permissive licensing only.
  - **Example:** *RefinedWeb* excludes paywalled or copyrighted domains.

- **Malware / Executable Detection (for code datasets):**
  - Sandbox or static analysis before including code examples.

### Techniques
- **Multi-layer filters:** Combine regex, heuristics, classifiers, and human spot-checks.
- **Scoring and Thresholding:** Each document gets a “safety score” → reject or flag borderline samples.
- **Policy metadata:** Unsafe but research-valuable data tagged instead of removed.

✅ *Advantages:* Increases model safety and compliance.  
❌ *Disadvantages:* Can overfilter legitimate edge cases; maintaining policy alignment over time is non-trivial.

---

## 4. Integrated Pipelines in Practice
| Project | Deduplication | Low-Quality Filtering | Safety Filtering |
|----------|----------------|-----------------------|------------------|
| **C4 (T5)** | LSH-based | Language + boilerplate filters | Basic profanity removal |
| **The Pile** | Hash + LSH | Per-source cleaning scripts | Minimal |
| **Dolma (AI2)** | Global hash + LSH | Model-based quality score | Safety classifiers |
| **RefinedWeb** | LSH + embedding | Quality score via LM | License & safety filters |
| **Phi-3 (Microsoft)** | Extensive | Readability + “textbookness” score | Filtered for instruction-like clarity |
| **Anthropic** | Embedding-based dedup | LM-based quality filter | Multi-category safety classifiers |

---

## 5. Key Takeaways
- **Deduplication:** Combines hash and embedding-based methods for scale and accuracy.  
- **Quality Filtering:** Shifting from static heuristics → LM-based scoring for nuanced curation.  
- **Safety Filtering:** Moving from rule-based → model-assisted moderation pipelines.  
- **Trend:** The best-performing LLMs (e.g., GPT-4, Claude 3, Phi-3) rely heavily on curated, high-quality, safe data—*quality, not size*, is the new scaling frontier.
