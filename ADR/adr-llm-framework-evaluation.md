# Architecture Decision Record (ADR) – Reasoning Enhancement Frameworks for LLMs

### 1. Title

Evaluating and Selecting Reasoning Enhancement Frameworks for Automated Grading and General AI Tasks

### 2. Context

To improve reasoning accuracy, explainability, and decision-making in automated grading and other AI-driven
applications, various frameworks have emerged. These frameworks enhance LLM outputs by refining, verifying, or
retrieving relevant information before finalizing responses. This ADR evaluates the most commonly used
reasoning-enhancement frameworks:

1. **Refine** – Iterative improvement of model outputs.
2. **ReAct (Reasoning + Acting)** – Interleaves reasoning with tool-use.
3. **Reflexion** – Self-reflective learning from past failures.
4. **Self-Verification** – Validates generated answers through independent re-evaluation.
5. **Chain-of-Thought (CoT)** – Breaks down reasoning into logical steps.
6. **Tree-of-Thought (ToT)** – Explores multiple solution paths before deciding.
7. **CoT + Refine + RAG** – Enhances structured reasoning while integrating retrieval-based verification.
8. **Agentic Frameworks (AutoGPT/BabyAGI, LangChain Agents)** – Multi-step adaptive AI reasoning.

### 3. Comparison of Reasoning Frameworks

| Framework                         | Purpose                                                                       | How It Works                                                                                                   | Best Use Cases                                                                | Limitations                                                           |
|-----------------------------------|-------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------|-----------------------------------------------------------------------|
| **Refine**                        | Improves response quality                                                     | Iteratively refines model outputs by improving previous outputs until a stable, high-quality answer is reached | Automated grading, QA, structured document generation                         | Can be slow due to multiple iterations and lacks external validation  |
| **ReAct (Reasoning + Acting)**    | Combines reasoning with external tool usage                                   | LLM generates thought processes and executes actions based on reasoning                                        | Research agents, interactive assistants, real-time API calls                  | High latency due to multiple interactions with external sources       |
| **Reflexion**                     | Self-improving model based on past failures                                   | Model reviews its own past incorrect responses and adjusts future answers                                      | AI coding assistants, iterative learning applications, autonomous AI research | Requires long-term memory or state-tracking, increasing complexity    |
| **Self-Verification**             | Ensures correctness by validating responses                                   | LLM independently evaluates its generated responses and adjusts if needed                                      | Automated grading, legal document validation, AI-based financial auditing     | Can reinforce existing biases and may struggle with conflicting logic |
| **Chain-of-Thought (CoT)**        | Enhances logical reasoning by step-by-step decomposition                      | Breaks complex problems into multiple logical steps for better clarity and structured answers                  | Math problem-solving, scientific explanations, structured assessments         | Still susceptible to factual inaccuracies if base reasoning is flawed |
| **Tree-of-Thought (ToT)**         | Explores multiple reasoning paths before selecting the best one               | Considers multiple potential solutions, evaluates each, and converges on the optimal response                  | Decision-making AI, strategic planning, multi-solution analysis               | Computationally expensive and requires additional evaluation layers   |
| **CoT + Refine + RAG**            | Enhances structured reasoning while integrating retrieval-based verification  | Leverages past graded responses to improve consistency                                                         | High-accuracy automated grading, AI-driven evaluation                         | Limited in handling edge cases with high grading variance             |
| **Multi-Agent Grading Framework** | Uses multiple specialized agents for grading, validation, and decision-making | One agent grades, another verifies, and a dean agent ensures fairness and final approval                       | High-stakes grading, multi-step AI workflows, regulatory decision-making      | High latency, complex orchestration, increased computational cost     |

### 4. Decision

We considered two primary approaches:

- **CoT + Refine + RAG (#7)** – This framework provides structured reasoning and retrieval-augmented verification,
  achieving **92% accuracy at 500ms latency**. While effective in improving consistency, it struggles with extreme
  variance in grading data and lacks a robust fairness mechanism.
- **Multi-Agent Grading Framework (#8)** – This approach introduces specialized agents (Grader, Evaluator, and Dean) to
  refine scores through multiple validation layers. It achieves 96% accuracy at the cost of increased inference
  time (1200ms per request). By isolating decision-making processes, it prevents bias accumulation and ensures fairness
  in grading.

There was significant debate regarding whether these approaches are fundamentally different or simply different
abstractions of structured reasoning. Both rely on iterative refinement and retrieval augmentation, but the multi-agent
framework introduces role-based separation to enforce unbiased validation and adjudication.

### 5. Selection and Justification

After evaluating multiple reasoning enhancement frameworks, we faced a decision between two closely related but distinct
approaches: CoT + Refine + RAG (#7) and the Multi-Agent Grading Framework (#8). Both leverage structured reasoning and
iterative refinement, but their implementation details lead to different strengths and trade-offs. The CoT + Refine +
RAG approach is highly effective at ensuring consistency through retrieval-augmented verification, achieving 92%
accuracy at 500ms latency. However, it lacks a robust fairness mechanism, particularly in cases where grading variance
is high. This creates a risk where a grader might learn incorrect score distributions based on a limited retrieved
dataset.

On the other hand, the Multi-Agent Grading Framework introduces specialized agents—Grader, Evaluator, and Dean—each
responsible for a distinct stage of assessment. By isolating these roles, it helps prevent graders from unintentionally
skewing scores based on incomplete or biased data. This approach ensures fairness by having different agents verify
scores separately rather than relying on a single model's judgment. It also resolves conflicts that arise when similar
answers receive widely different scores due to inconsistencies in human-graded examples, leading to a more balanced and
justified evaluation process, ultimately improving accuracy to 96% at the cost of increased inference time (1200ms per
request). Though initially debated, we recognized that this agentic approach is not fundamentally different from CoT +
Refine + RAG in structure but rather offers greater control over how validation and adjudication are handled. It ensures
that grading is not influenced by incomplete data retrieval while still benefiting from iterative refinement.

Given the trade-offs, we selected the Multi-Agent Grading Framework (#8) for automated grading due to its higher
accuracy, structured fairness mechanisms, and superior scalability for future enhancements. While CoT + Refine + RAG
remains a viable alternative, its limitations in handling extreme variance made it less suitable for our use case.

### 6. Multi-Agent Grading Framework Consideration

#### Agent Roles and Functions

| **Agent**           | **Function**                                                                     |
|---------------------|----------------------------------------------------------------------------------|
| **Grader Agent**    | Scores the response based on the rubric (**CoT + Refine**)                       |
| **Evaluator Agent** | Cross-checks the score against SME-graded examples (**RAG + Self-Verification**) |
| **Dean Agent**      | Resolves conflicts, applies final refinements, and ensures fairness              |

#### Benefits of Multi-Agent Grading Framework:

- Higher accuracy due to specialized agents focusing on different aspects of grading.
- Better consistency by separating scoring from evaluation.
- Improved fairness by ensuring that grading variance is considered across multiple SME data points.

### 7. Bias & Variance Considerations

- **Bias Risk:** If the Grader sees past scores via RAG, it might drift toward an incorrect average score.
- **Variance Awareness:** The Evaluator and Dean ensure that the Grader’s score aligns with acceptable variance levels.
- **Correction Mechanism:** Evaluator nudges the Grader in low-variance cases (3-5 scores) while maintaining
  fairness in high-variance cases (1-5 scores).

### 8. Conclusion

The **Multi-Agent Grading Framework with RAG and Refinement** was selected due to its modularity,
high accuracy, and fairness in handling grading variance. The Evaluator acts as an independent correction mechanism,
while the Dean ensures final arbitration. Future improvements could involve adaptive retrieval, confidence-weighted
scoring, or hierarchical review layers to further refine accuracy and efficiency.

