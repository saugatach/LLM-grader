# Architecture Decision Record (ADR) – Automated Architecture Exam Grading System

### **1. Title**
**Automated Architecture Exam Grading System with OpenAI LLM and Self-Evaluation Frameworks**

### **2. Context**

The goal is to design an automated grading system for an **architecture certification exam**, where answers are **free-form, short responses (<200 words)**. Grading must be on a **5-point scale**, and we have a dataset of **200,000 SME-graded responses**. 

Initial considerations included **fine-tuning an open-source LLM (e.g., LLaMA 3, DeepSeek MoE)**, but the **cost of training and deployment was too high**. Instead, the decision is to use **a commercially available LLM (OpenAI GPT-4-turbo)** for **batch-based grading**, optimizing for **accuracy over latency** since real-time processing is not required. 

To **improve accuracy and consistency**, **self-evaluation frameworks** such as **Chain-of-Thought (CoT), Refine, and Retrieval-Augmented Generation (RAG)** are integrated. 

### **3. Decision**

We will implement a **hybrid AI grading pipeline** that uses:
1. **OpenAI GPT-4-turbo** for **initial scoring**.
2. **Self-Evaluation via CoT + Refine** to **verify and improve grading consistency**.
3. **RAG-based Justification** to retrieve **past graded examples from a vector database to provide explainability** by showing candidates how similar answers were graded in the past.
4. **Batch Processing Pipeline** to optimize cost and throughput, as the high latency of the pipeline is a tradeoff for achieving high accuracy. Due to this, real-time processing is impractical, and the system must be executed as a batch job.
5. **Fallback Mechanisms** for SME intervention in edge cases. Additionally, Reinforcement Learning from Human Feedback (RLHF) could be explored to iteratively improve grading accuracy by incorporating SME feedback directly into model training.

### **4. Comparison of Approaches**

| Approach | LLM Cost (Fine-tuning & Hosting) | Inference Cost per Request | Accuracy (%) | Latency (ms) | Explainability |
|-----------|------------------------------|---------------------------|--------------|--------------|----------------|
| **Option 1: Fine-Tuned LLaMA 3 / DeepSeek MoE** | High ($10K–$50K) | ~$0.01 | 95% | 300ms | Low (No built-in justification) |
| **Option 2: OpenAI GPT-4 Basic Few-Shot Prompting** | None | ~$0.004 | 85% | 200ms | Low (No self-verification) |
| **Option 3: OpenAI GPT-4 + CoT + Refine + RAG** | None | ~$0.006–$0.008 | 92% | 500ms | High (Retrieves past graded responses) |

### **5. System Architecture**
#### **Components & Workflow**
1. **Grading Engine:**
   - OpenAI GPT-4-turbo scores responses with **few-shot prompting.**
   - Batch processing ensures **scalability and cost control.**

2. **RAG-Based Justification:**
   - Retrieves similar past graded responses from a vector database.
   - Provides contextual examples to justify scoring decisions.
   - Ensures that candidates understand why they received a specific grade.
   
3. **Accuracy Improvement Framework:**
   - **CoT Reasoning** ensures **step-by-step scoring validation**.
   - **Refine Framework** iterates over the score to **minimize inconsistencies**.

4. **Scalability & Cost Optimization:**
   - **Batch execution** for cost efficiency.
   - **Confidence thresholding:** High-confidence scores are accepted; low-confidence ones are **reviewed manually**.
   - **Data caching** for redundant responses.

5. **User Interface & API:**
   - **Backend:** FastAPI-based API for batch grading.
   - **Frontend:** React-based UI with **explanations & SME override options**.
   - **RAG UI Module**: Displays past graded examples for candidates and SMEs.

### **6. Alternative Enhancements Considered**

| Enhancement | LLM Cost | Inference Cost per Request | Accuracy | Latency | Pros | Cons |
|-------------|----------|---------------------------|----------|---------|------|------|
| **Fine-Tuned Distilled Model (TinyLLaMA / Phi-2)** | Medium ($5K–$15K) | Low ($0.002–$0.005) | Moderate (88%) | 150ms | Lower inference cost, faster response times | Requires additional training, lower accuracy than GPT-4 |
| **Hybrid Strategy (GPT-4 Primary, Fine-Tuned Model as Fallback)** | Medium ($5K–$20K) | Medium ($0.006 per request avg.) | High (94%) | 400ms | Optimizes for cost and performance, balances trade-offs | More complex implementation, needs routing logic |
| **Edge Case Handling with Human-in-the-Loop (HITL)** | None (SME costs apply) | High (Manual review required) | Very High (98%) | Variable | Ensures human oversight for ambiguous cases | Slower, requires SME intervention |

### **7. Consequences**
✅ **Pros:**
- **Cost-effective** (~$0.006 per request vs. $0.01 for fine-tuning LLaMA 3).
- **Higher accuracy** than naive prompting.
- **Explainability and fairness** via RAG justification.
- **No infrastructure management required.**

⚠️ **Challenges & Mitigation Strategies:**
- **Latency (500ms per request)** → Not an issue due to **batch processing**.
- **Reliance on OpenAI API** → Mitigated by potential **backup open-source model**.
- **RAG retrieval relevance** → Requires **tuning for domain-specific grading**.

### **8. Next Steps**
1. **Prototype GPT-4 grading pipeline** with **real SME responses**.
2. **Evaluate CoT + Refine impact** on accuracy consistency.
3. **Integrate RAG for Justification UI.**
4. **Test batch execution for cost efficiency.**
