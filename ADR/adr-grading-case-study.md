## Architecture Decision Record (ADR) – Automated Grading System for Case Studies with Multimodal Processing

### 1. Title
**Evaluating and Selecting an Agentic Multi-Agent Framework for Case Study Grading with Text and Diagram Analysis**

### 2. Context

The goal is to design an automated grading system for **architecture case studies**, where answers include **long-form text, citations, and architecture diagrams**. Unlike short responses, case studies require evaluation of multiple dimensions:

1. **Structured Text Evaluation** – Assessing different sections (problem statement, methodology, conclusions, etc.) independently while maintaining overall coherence.
2. **Reference Verification** – Checking if references are correctly cited, relevant, and substantiate the case study.
3. **Architecture Diagram Processing** – Understanding and validating visual components to ensure alignment with the textual explanation.
4. **Handling Long Contexts** – Managing context length limitations by implementing chunking and retrieval-based summarization.

Previous grading methods for short responses relied on **CoT + Refine + RAG** or a **Multi-Agent Grading Framework** for structured evaluation. Given the added complexities, we need an improved approach that integrates **multimodal processing** while maintaining **scalability, explainability, and fairness** in grading.

### 3. Decision

For case study grading, the ideal approach should:
1. **Improve accuracy** by ensuring that all case study components (text, references, diagrams) are evaluated holistically.
2. **Handle long-form responses efficiently** using a combination of hierarchical summarization and retrieval-based chunking.
3. **Provide explainability** by making grading justifications transparent at both the section and final score levels.
4. **Integrate multimodal processing** to evaluate architecture diagrams alongside textual content.
5. **Balance performance and cost** by choosing the right architecture for diagram analysis.

### 4. Selected Approach
- **Multi-Agent Framework with Multimodal Processing:**
  - **Text Grader Agent:** Evaluates different sections independently using **CoT + Refine**.
  - **Reference Checker Agent:** Validates citations and checks reference integrity.
  - **Diagram Evaluator Agent:** Uses a **multimodal LLM (e.g., GPT-4V, Gemini)** to analyze architecture diagrams and ensure alignment with the case study.
  - **Final Aggregator (Dean Agent):** Merges partial scores, applies section-based weighting, and ensures fairness in grading. The Dean Agent includes a threshold mechanism to detect inconsistent scores and escalate ambiguous cases for additional validation, ensuring grading fairness and reliability.

### 5. Evaluating Diagram Processing Methods
We considered two primary approaches for analyzing architecture diagrams:

1. **Using a Fully Multimodal Model (GPT-4V, Gemini, OpenFlamingo)**
   - **Pros:** Direct processing of both text and images, higher accuracy in contextual analysis.
   - **Cons:** Higher inference costs, limited control over extracted diagram descriptions.

2. **Using LLaVA (or Similar) for Diagram Interpretation + Text-Based LLM for Analysis**
   - **Pros:** More modular, allows separate control over visual and textual grading, more cost-effective.
   - **Cons:** Requires additional processing steps and alignment between vision and text outputs.

**Final Decision:**
- **If accuracy is the priority, a fully multimodal model (GPT-4V, Gemini) is preferred.**
- **If cost and modularity are key, a two-step process using LLaVA for diagram extraction followed by a text-based LLM for analysis is better.**
- Given the case study’s complexity, we prioritize accuracy over cost, selecting a fully multimodal model for diagram evaluation. To mitigate costs, we will implement selective multimodal processing where diagrams are processed only when necessary, and batch inference strategies will be used to optimize performance.

### 6. Updated Cost Considerations
The integration of multimodal capabilities increases token usage and requires **updated cost estimates**:

| **Approach** | **LLM Cost (Fine-tuning & Hosting)** | **Inference Cost per Request** | **Accuracy (%)** | **Latency (ms)** | **Explainability** |
|-------------|--------------------------------|---------------------------|--------------|--------------|----------------|
| **Option 1: Fine-Tuned LLaMA 3 / DeepSeek MoE** | High ($20K+) | ~$0.012 | 95% | 500ms | Low (No built-in justification) |
| **Option 2: OpenAI GPT-4 Basic Few-Shot Prompting** | None | ~$0.006 | 85% | 300ms | Low (No self-verification) |
| **Option 3: GPT-4 + CoT + Refine + RAG** | None | ~$0.008 | 92% | 700ms | High (Retrieves past graded responses) |
| **Option 4: Multi-Agent Framework + Multimodal LLM (GPT-4V, Gemini)** | None | ~$0.018 (4x LLM calls) | 97% | 1500ms | Very High (Text + Diagram Verification). Breakdown of LLM calls: (1) Initial grading pass for text-based sections, (2) Reference validation and retrieval augmentation, (3) Multimodal evaluation of architecture diagrams, (4) Final aggregation and score normalization. Future optimizations may include selectively invoking multimodal processing only when diagrams are detected, reducing unnecessary computation.

### 7. Image Processing Challenges and Fallback Mechanism

Processing architecture diagrams introduces complexities due to variations in diagram styles, text annotations, and model limitations. While multimodal models are effective, they are not foolproof. To address failures in recognizing and extracting information, we implement a fallback mechanism:

**Introduce a confidence threshold:**
- **High-confidence multimodal extraction** → Normal pipeline.
- **Low-confidence multimodal extraction** → Falls back to OCR + Rule-Based Parsing.
- **Very low confidence** → LLaVA-generated textual description.
- **Failed extraction completely** → Escalate to Human-in-the-Loop (HITL).

To ensure reliable processing, we establish a structured fallback strategy:

1. **Confidence-Based Rerouting:** If the multimodal model outputs a low-confidence score, we trigger alternative processing steps.
2. **OCR + Rule-Based Parsing:** If image recognition fails, an OCR-based approach extracts text labels and relationships to construct a structured representation.
3. **Diagram-to-Text with LLaVA:** If OCR parsing is insufficient, a vision-language model like LLaVA generates textual descriptions of the diagram, which are then analyzed using a text-based LLM.
4. **Human-in-the-Loop (HITL) Escalation:** If all automated methods fail, the system flags the diagram for manual review, ensuring accuracy in grading.

While these fallback methods increase processing costs, they are still significantly lower than paying human evaluators at $35/hour. Additionally, selective multimodal invocation and confidence-based decision-making help optimize expenses.

### 8. System Architecture Considerations

1. **Text Evaluation**
   - **Chunking and Hierarchical Summarization** for handling long contexts.
   - **CoT + Refine** for structured reasoning within each section.
   - **RAG for Reference Checking** to compare citations with a knowledge base.

2. **Multimodal Processing for Diagrams**
   - **OCR and Graph Extraction** for structured analysis.
   - **Multimodal LLM** for evaluating architectural correctness and consistency.

3. **Agent-Based Grading System**
   - **Text Grader Agent** → Scores different case study sections.
   - **Reference Checker Agent** → Validates citations and reference accuracy.
   - **Diagram Evaluator Agent** → Uses a multimodal model to analyze diagrams.
   - **Final Aggregator (Dean Agent)** → Ensures overall grading fairness.

### 8. Alternative Enhancements Considered

| **Enhancement** | **Benefits** | **Trade-offs** |
|-------------|----------|-------------|
| **Fine-Tuned Multimodal Model** | Highest accuracy, tailored for case study grading | Expensive, requires dedicated training |
| **Hybrid GPT-4V + Open-Source Vision Model** | Cost-effective, balances accuracy and cost | Complex integration required |
| **Human-in-the-Loop (HITL) for Review** | Ensures fairness in ambiguous cases | Slower, SME intervention required |

### 9. Consequences
✅ **Pros:**
- **Higher accuracy (97%)** through text + diagram evaluation.
- **Explainability significantly improved** by separating grading responsibilities.
- **Better handling of long responses** using hierarchical processing.

⚠️ **Challenges & Mitigation Strategies:**
- **Higher latency (1500ms per request)** → Batch processing avoids real-time performance issues.
- **Computational Cost (~3x inference cost increase)** → Optimized multimodal calls to reduce expenses.

### 10. Conclusion
For grading case studies, a **Multi-Agent Grading Framework with Multimodal LLM Integration** was selected due to its superior accuracy, structured evaluation, and ability to process both text and diagrams holistically. Future improvements may involve **hybrid approaches** to balance cost and accuracy, such as selectively applying multimodal processing only when diagrams are present. Further enhancements could include **adaptive retrieval strategies for reference validation** and **confidence-weighted scoring** to refine accuracy and fairness.
