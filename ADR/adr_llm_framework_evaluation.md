**Architecture Decision Record (ADR) – Reasoning Enhancement Frameworks for LLMs**

### **1. Title**
**Evaluating and Selecting Reasoning Enhancement Frameworks for Automated Grading and General AI Tasks**

### **2. Context**

To improve reasoning accuracy, explainability, and decision-making in **automated grading and other AI-driven applications**, various frameworks have emerged. These frameworks **enhance LLM outputs** by refining, verifying, or retrieving relevant information before finalizing responses. This ADR evaluates the most commonly used reasoning-enhancement frameworks:

1. **Refine** – Iterative improvement of model outputs.
2. **ReAct (Reasoning + Acting)** – Interleaves reasoning with tool-use.
3. **Reflexion** – Self-reflective learning from past failures.
4. **Self-Verification** – Validates generated answers through independent re-evaluation.
5. **Chain-of-Thought (CoT)** – Breaks down reasoning into logical steps.
6. **Tree-of-Thought (ToT)** – Explores multiple solution paths before deciding.
7. **Agentic Frameworks (AutoGPT/BabyAGI, LangChain Agents)** – Multi-step adaptive AI reasoning.

### **3. Decision**

For **automated grading**, the ideal approach should:
1. **Improve accuracy** by reducing hallucinations and inconsistencies.
2. **Provide explainability** for grading justifications.
3. **Balance latency and cost-effectiveness** for batch processing.

**Selected Approach:**
- **Primary Framework:** Multi-agent framework where all agents utilize **Refine + CoT** for structured reasoning, with the **Evaluator Agent exclusively augmented with RAG** to verify score consistency against SME-graded examples. RAG enhances explainability by retrieving past graded responses, ensuring alignment with prior evaluations.
- **Selected Approach:** **Agentic framework with a Grader Agent, Evaluator Agent, and Dean Agent, all utilizing CoT and Refine**. This multi-agent setup ensures specialized grading, verification, and fairness resolution, leading to **higher accuracy and explainability**.

### **4. Comparison of Reasoning Frameworks**

| Framework | Purpose | How It Works | Best Use Cases | Limitations |
|-----------|---------|-------------|---------------|-------------|
| **Refine** | Improves response quality | Iteratively refines model outputs by improving previous outputs until a stable, high-quality answer is reached | Automated grading, QA, structured document generation | Can be slow due to multiple iterations and lacks external validation |
| **ReAct (Reasoning + Acting)** | Combines reasoning with external tool usage | LLM generates thought processes and executes actions based on reasoning | Research agents, interactive assistants, real-time API calls | High latency due to multiple interactions with external sources |
| **Reflexion** | Self-improving model based on past failures | Model reviews its own past incorrect responses and adjusts future answers | AI coding assistants, iterative learning applications, autonomous AI research | Requires long-term memory or state-tracking, increasing complexity |
| **Self-Verification** | Ensures correctness by validating responses | LLM independently evaluates its generated responses and adjusts if needed | Automated grading, legal document validation, AI-based financial auditing | Can reinforce existing biases and may struggle with conflicting logic |
| **Chain-of-Thought (CoT)** | Enhances logical reasoning by step-by-step decomposition | Breaks complex problems into multiple logical steps for better clarity and structured answers | Math problem-solving, scientific explanations, structured assessments | Still susceptible to factual inaccuracies if base reasoning is flawed |
| **Tree-of-Thought (ToT)** | Explores multiple reasoning paths before selecting the best one | Considers multiple potential solutions, evaluates each, and converges on the optimal response | Decision-making AI, strategic planning, multi-solution analysis | Computationally expensive and requires additional evaluation layers |
| **Multi-Agent Grading Framework** | Uses multiple specialized agents for grading, validation, and decision-making | One agent grades, another verifies, and a dean agent ensures fairness and final approval | High-stakes grading, multi-step AI workflows, regulatory decision-making | High latency, complex orchestration, increased computational cost |

### **5. System Architecture Considerations**

1. **For Batch-Based Grading:**
   - **Refine + Self-Verification + RAG** ensures stable and consistent scoring.
   - **Low-cost inference and moderate latency** are acceptable since **real-time performance is not required**.

2. **For Real-Time AI Agents:**
   - **ReAct + Reflexion** provides tool integration and adaptability.
   - **High cost and latency trade-offs** need optimization.

3. **For Scientific and Analytical Reasoning:**
   - **CoT or ToT** enables structured thinking and better problem-solving.

### **6. Alternative Enhancements Considered**

| Enhancement | Benefits | Trade-offs |
|-------------|----------|-------------|
| **Fine-Tuned LLaMA 3 with RLHF** | Higher accuracy, customized grading model | Expensive, requires retraining |
| **Hybrid GPT-4 + Open-Source Models** | Cost-effective, balances accuracy and cost | Complex routing logic needed |
| **Human-in-the-Loop (HITL) for Review** | Ensures fairness in ambiguous cases | Slower, SME intervention required |

### **7.1 Multi-Agent Grading Framework Consideration**

An alternative approach to improve grading accuracy is to adopt an **agentic framework** where multiple specialized agents collaborate on the grading process. This framework introduces the following roles:

| **Agent**  | **Function** |
|------------|-------------|
| **Grader Agent** | Scores the response based on the rubric (**CoT + Refine**) |
| **Evaluator Agent** | Cross-checks the score against SME-graded examples (**RAG + Self-Verification**) |
| **Dean Agent** | Resolves conflicts, applies final refinements, and ensures fairness |

#### **Benefits of Multi-Agent Grading Framework:**
- **Higher accuracy** due to specialized agents focusing on different aspects of grading.
- **Better consistency** by separating scoring from evaluation.
- **Explainability** is improved as decisions are reviewed at multiple levels.

