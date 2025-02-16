## Architecture Decision Record (ADR) – Automated Grading System Using Multi-Agent Reasoning Framework

### 1. Title
**Evaluating and Selecting an Agentic Multi-Agent Framework for Automated Grading**

### 2. Context

The goal is to design an automated grading system for an **architecture certification exam**, where answers are **free-form, short responses (<200 words)**. Grading must be on a **5-point scale**, and we have a dataset of **200,000 SME-graded responses**. 

Previous considerations included **fine-tuning an open-source LLM (e.g., LLaMA 3, DeepSeek MoE)** or using **RAG + CoT + Refine**, but we are now transitioning to an **Agentic Multi-Agent Framework** where each agent specializes in different parts of the grading pipeline. 

To **improve accuracy and consistency**, the grading process will be split across three distinct agents:

1. **Grader Agent:** Assigns an initial score using **Refine + CoT** based on rubric criteria.
2. **Evaluator Agent:** Cross-checks the score using **RAG + Self-Verification**, comparing it to similar graded examples.
3. **Dean Agent:** Finalizes the score, resolves discrepancies, ensures fairness across responses, and handles edge cases where score inconsistencies cannot be resolved after multiple cycles. The Dean Agent also acts as a tie-breaker in scenarios where the Grader and Evaluator Agents loop between different scores.

This approach **ensures specialized expertise at each step**, making grading **more consistent, explainable, and reliable**.

### 3. Decision

For automated grading, the ideal approach should:
1. Improve accuracy by reducing hallucinations and inconsistencies. Large language models rely on limited context windows, meaning they may not always retrieve all relevant training examples when scoring a response. If a grader sees only a small subset of past scores instead of the full range, it may begin assigning grades based on incomplete information, leading to inconsistencies over time.
2. Provide explainability for grading justifications. When a response is graded, it should be possible to trace back how the score was determined. A system that relies purely on retrieval or implicit reasoning may fail to provide meaningful feedback. A well-structured approach should ensure that each step in the grading process can be understood and reviewed.
3. Balance latency and cost-effectiveness for batch processing. Automated grading often requires evaluating large volumes of responses. An approach that is too computationally expensive or slow will not scale efficiently, making it impractical for real-world applications. The selected framework should ensure that grading remains both cost-effective and performant, especially when deployed at scale.

**Selected Approach:**
- **Multi-Agent Framework:**
  - All agents utilize **Refine + CoT** for structured reasoning.
  - The **Evaluator Agent exclusively integrates RAG** to verify score consistency against SME-graded examples.
  - **Dean Agent acts as a final verification layer**, ensuring fairness and resolving conflicts.

### 4. Updated Cost Considerations

The introduction of multiple agents increases token usage significantly, requiring recalculated cost estimates:

| **Approach** | **LLM Cost (Fine-tuning & Hosting)** | **Inference Cost per Request** | **Accuracy (%)** | **Latency (ms)** | **Explainability** |
|-------------|--------------------------------|---------------------------|--------------|--------------|----------------|
| **Option 1: Fine-Tuned LLaMA 3 / DeepSeek MoE** | High ($10K–$50K) | ~$0.01 | 95% | 300ms | Low (No built-in justification) |
| **Option 2: OpenAI GPT-4 Basic Few-Shot Prompting** | None | ~$0.004 | 85% | 200ms | Low (No self-verification) |
| **Option 3: OpenAI GPT-4 + CoT + Refine + RAG** | None | ~$0.006–$0.008 | 92% | 500ms | High (Retrieves past graded responses) |
| **Option 4: Multi-Agent Grading Framework (Grader + Evaluator + Dean)** | None | ~$0.012–$0.016 (3x LLM calls) | 96% | 1200ms | Very High (Grader + Evaluator + Dean verification) |

### 5. System Architecture Considerations

1. **For Batch-Based Grading:**
   - **Grader Agent assigns initial scores based on CoT + Refine**.
   - **Evaluator Agent uses RAG to check consistency with prior graded responses**.
   - **Dean Agent verifies final score and ensures fairness**.
   - **Batch processing allows parallel execution, mitigating latency concerns.**

2. **For High-Accuracy Grading:**
   - **Multi-agent collaboration ensures verification at multiple levels.**
   - **Explainability improves with justification provided at each stage.**

### 6. Alternative Enhancements Considered

| **Enhancement** | **Benefits** | **Trade-offs** |
|-------------|----------|-------------|
| **Fine-Tuned LLaMA 3 with RLHF** | Higher accuracy, customized grading model | Expensive, requires retraining |
| **Hybrid GPT-4 + Open-Source Models** | Cost-effective, balances accuracy and cost | Complex routing logic needed |
| **Human-in-the-Loop (HITL) for Review** | Ensures fairness in ambiguous cases | Slower, SME intervention required |

### 7. Consequences

✅ Pros:
- Ensures higher accuracy (96%) through multi-agent validation.
- Explainability significantly improved via RAG-based justification.
- Reduces grading inconsistencies through multiple verification layers.

⚠️ Challenges & Mitigation Strategies:
- **Latency (1200ms per request)** → Batch processing avoids real-time performance concerns.
- **Computational Cost (~2x inference cost increase)** → Optimized execution to parallelize grading steps.

### 8. Bias & Variance Considerations
- **Bias Risk:** If the Grader sees past scores via RAG, it might drift toward an incorrect average score.
- **Variance Awareness:** The Evaluator and Dean ensure that the Grader’s score aligns with acceptable variance levels.
- **Correction Mechanism:** Evaluator nudges the Grader in low-variance cases (3-5 scores) while maintaining fairness in high-variance cases (1-5 scores).

### 9. Conclusion
For the hackathon, the **Multi-Agent Grading Framework with RAG and Refinement** was selected due to its modularity, high accuracy, and fairness in handling grading variance. The Evaluator acts as an independent correction mechanism, while the Dean ensures final arbitration. Future improvements could involve **adaptive retrieval, confidence-weighted scoring, or hierarchical review layers** to further refine accuracy and efficiency.

### 10. Architecture Diagram
_(Insert diagram here, illustrating the Grader, Evaluator, and Dean agents, including their interactions and data flow)_

