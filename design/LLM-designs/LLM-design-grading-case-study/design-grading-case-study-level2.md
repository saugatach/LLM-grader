x# Case Study Grading System - Level 2 Technical Architecture

## **1. Introduction**
### **1.1 Purpose**
This document provides a **detailed Level 2 architecture** for the Case Study Grading System, focusing on:
- **Detailed component interactions and API-level design**
- **Orchestration of grading processes and workflows**
- **Implementation details for developers**
- **Processing pipelines and execution logic**

This architecture assumes a **single model approach using GPT-4** for all grading components.

---

## **2. System Components and Orchestration**
### **2.1 Core Components**
1. **Submission Handler** – Receives case study documents (text, references, diagrams).
2. **Text Processing Module** – Splits long-form text into structured sections for evaluation.
3. **Reference Validation Module** – Extracts and validates citations using RAG.
4. **Diagram Processing Module** – Uses multimodal processing to evaluate architecture diagrams.
5. **Scoring and Aggregation Module** – Applies weights and normalizes scores.
6. **Feedback Generator** – Constructs a structured feedback report for users.
7. **Workflow Orchestrator** – Manages the end-to-end process execution.

---

## **3. System Interaction Patterns**

### **3.1 Component Interaction Sequence**
```mermaid
sequenceDiagram
    participant User
    participant SubmissionHandler as Submission Handler
    participant TextProcessor as Text Processing Module
    participant ReferenceValidator as Reference Validation Module
    participant DiagramProcessor as Diagram Processing Module
    participant ScoringAggregator as Scoring & Aggregation Module
    participant FeedbackGenerator as Feedback Generator
    participant Orchestrator as Workflow Orchestrator

    User->>SubmissionHandler: Upload Case Study (Text + References + Diagrams)
    SubmissionHandler->>Orchestrator: Validate & Initiate Processing
    
    par Process Text
        Orchestrator->>TextProcessor: Extract and Evaluate Text Sections
    and Process References
        Orchestrator->>ReferenceValidator: Verify Citations and References
    and Process Diagrams
        Orchestrator->>DiagramProcessor: Analyze and Score Diagrams
    end

    TextProcessor-->>ScoringAggregator: Return Evaluated Text Scores
    ReferenceValidator-->>ScoringAggregator: Return Reference Validation Scores
    DiagramProcessor-->>ScoringAggregator: Return Diagram Scores
    
    ScoringAggregator->>FeedbackGenerator: Generate Feedback Report
    FeedbackGenerator->>User: Provide Grading Report
```

### **3.2 System Boundary Context**
```mermaid
graph TB
    subgraph External Systems
        A[User Interface]
        B[Authentication Service]
        C[Storage Service]
    end

    subgraph Grading System
        subgraph Processing Layer
            D[Submission Handler]
            E[Text Processor]
            F[Reference Validator]
            G[Diagram Processor]
        end
        
        subgraph Core Layer
            H[Workflow Orchestrator]
            I[Scoring Aggregator]
        end
        
        subgraph AI Layer
            J[GPT-4V Service]
            K[LLaVA Service]
            L[OCR Service]
        end
    end

    subgraph External Services
        M[HITL Review System]
        N[Monitoring System]
    end

    A --> D
    D --> H
    H --> E & F & G
    E & F & G --> I
    E & G --> J
    G --> K
    G --> L
    G --> M
```

### **3.3 Data Flow Patterns**
```mermaid
flowchart TD
    subgraph Input
        A[Case Study Document] --> B[Document Parser]
        B --> C[Content Extractor]
    end

    subgraph Processing
        C --> D[Text Content]
        C --> E[References]
        C --> F[Diagrams]
        
        D --> G[Text Analysis]
        E --> H[Reference Validation]
        F --> I[Image Processing]
        
        G --> J[Text Scores]
        H --> K[Reference Scores]
        I --> L[Diagram Scores]
    end

    subgraph Aggregation
        J --> M[Score Aggregator]
        K --> M
        L --> M
        M --> N[Final Score]
    end

    subgraph Output
        N --> O[Feedback Generator]
        O --> P[Grading Report]
    end
```

### **3.4 Component Communication Patterns**
```mermaid
graph TD
    subgraph Synchronous Operations
        A[API Gateway]
        B[Submission Handler]
        C[Workflow Orchestrator]
        A --> B
        B --> C
    end

    subgraph Asynchronous Operations
        D[Message Queue]
        E[Text Processor]
        F[Reference Validator]
        G[Diagram Processor]
        C --> D
        D --> E & F & G
    end

    subgraph Event-Driven
        H[Event Bus]
        I[Score Aggregator]
        J[Feedback Generator]
        E & F & G --> H
        H --> I
        I --> J
    end

    style Event-Driven fill:#e6e6ff,stroke:#333,stroke-width:2px
```

### **3.5 Interface Definitions**

1. **External Interfaces**
   - User Interface: REST/HTTP
   - Authentication Service: OAuth2/OIDC
   - Storage Service: S3-compatible API
   - HITL Review System: Event-driven/Webhooks

2. **Internal Component Interfaces**
   - Processing Layer: gRPC
   - Core Layer: Message Queue (RabbitMQ/Kafka)
   - AI Service Integration: REST/HTTP
   - Event Bus: Pub/Sub Pattern

3. **Communication Protocols**
   - Synchronous: HTTP/2, gRPC
   - Asynchronous: AMQP, Kafka Protocol
   - Events: CloudEvents specification

## **4. Execution Flow & Processing Pipelines**

### **4.1 Grading Pipeline**
```mermaid
sequenceDiagram
    participant S as Submission Service
    participant O as Orchestrator
    participant T as Text Processor
    participant R as Reference Checker
    participant D as Diagram Processor
    participant A as Score Aggregator
    participant F as Feedback Generator

    S->>O: Submit Case Study
    activate O
    O->>O: Validate & Extract Components
    
    par Text Processing
        O->>T: Process Text Sections
        activate T
        T-->>O: Text Evaluation Complete
        deactivate T
    and Reference Processing
        O->>R: Validate References
        activate R
        R-->>O: Reference Check Complete
        deactivate R
    and Diagram Processing
        O->>D: Process Diagrams
        activate D
        D->>D: Image Verification
        D->>D: Confidence Check
        alt High Confidence
            D->>D: Direct GPT-4V Analysis
        else Medium Confidence
            D->>D: OCR + Rule Processing
        else Low Confidence
            D->>D: LLaVA Description
        end
        D-->>O: Diagram Analysis Complete
        deactivate D
    end

    O->>A: All Components Processed
    deactivate O
    activate A
    A->>F: Generate Report
    deactivate A
    activate F
    F-->>S: Return Grading Report
    deactivate F
```

### **4.2 Image Verification Pipeline**
```mermaid
flowchart TD
    A[Case Study Submission] --> B[Image Extraction Service]
    B --> |Extract Images| C[Metadata Analysis]
    
    subgraph Metadata["Metadata Processing"]
        C --> |Check| D[Format Validation]
        C --> |Analyze| E[Resolution Check]
        C --> |Verify| F[Size Constraints]
    end
    
    Metadata --> G[Initial GPT-4V Analysis]
    G --> H{Confidence Score Assessment}
    
    H -->|Score > 0.85| I[High Confidence Path]
    H -->|0.60-0.85| J[Medium Confidence Path]
    H -->|0.30-0.60| K[Low Confidence Path]
    H -->|< 0.30| L[Very Low Confidence Path]
    
    subgraph HighConf["Direct Processing"]
        I --> I1[GPT-4V Analysis]
        I1 --> I2[Architecture Validation]
        I2 --> I3[Generate Score]
    end
    
    subgraph MedConf["OCR Processing"]
        J --> J1[OCR Text Extraction]
        J1 --> J2[Rule-Based Parsing]
        J2 --> J3[Generate Score]
    end
    
    subgraph LowConf["LLaVA Processing"]
        K --> K1[Generate Description]
        K1 --> K2[Text Analysis]
        K2 --> K3[Generate Score]
    end
    
    subgraph VLowConf["HITL Escalation"]
        L --> L1[Queue for Review]
        L1 --> L2[Expert Review]
        L2 --> L3[Manual Score]
    end
    
    HighConf & MedConf & LowConf & VLowConf --> M[Score Aggregation]
    M --> N[Final Evaluation]
    
    style HighConf fill:#e6ffe6,stroke:#333,stroke-width:2px
    style MedConf fill:#fff5e6,stroke:#333,stroke-width:2px
    style LowConf fill:#ffe6e6,stroke:#333,stroke-width:2px
    style VLowConf fill:#e6e6ff,stroke:#333,stroke-width:2px
    style Metadata fill:#f0f0f0,stroke:#333,stroke-width:2px
```

### **4.3 Processing Steps**
1. **Document Submission & Preprocessing**
   - Extract text, references, and diagrams
   - Validate submission format and structure
2. **Text Processing & Evaluation**
   - Use GPT-4 to analyze sections based on rubrics
   - Apply CoT + Refine to improve accuracy
3. **Reference Checking**
   - Retrieve cited references
   - Compare with case study content
4. **Diagram Evaluation**
   - Process images using GPT-4V
   - Apply confidence-based routing
5. **Score Aggregation**
   - Normalize scores from different modules
   - Apply section weightings
6. **Feedback Generation**
   - Structure justifications
   - Provide actionable feedback

---

## **5. API & Interface Design**
| **Component** | **API Endpoint** | **Method** | **Description** |
|--------------|----------------|------------|----------------|
| **Submission Handler** | `/submit` | `POST` | Upload case study for grading |
| **Text Processor** | `/process-text` | `POST` | Process and analyze text sections |
| **Reference Validator** | `/validate-references` | `POST` | Verify accuracy of citations |
| **Diagram Processor** | `/process-diagram` | `POST` | Analyze architecture diagrams |
| **Scoring Aggregator** | `/aggregate-scores` | `POST` | Normalize and compute final score |
| **Feedback Generator** | `/generate-feedback` | `POST` | Generate structured feedback report |

---

## **6. High Availability & Fault Tolerance**

```mermaid
graph TD;
    A[Case Study Submission] -->|Text| B[Text Grader]
    A -->|References| C[Reference Checker]
    A -->|Diagrams| D[Diagram Evaluator]
    B --> E[Final Aggregator]
    C --> E
    D --> E
    E -->|Final Score| F[Grading Report]
    E -->|Failover Trigger| G[Redundant Processing Pipeline]
    G -->|Reprocess| E
    D -->|Low Confidence| H[LLaVA-generated Descriptions]
    H -->|Extracted Text| D
    H -->|Very Low Confidence| I[OCR-based Parsing]
    I -->|Generated Text| D
    I -->|Complete Failure| J[Human-in-the-loop Escalation]
```

### **6.1 Resilience Mechanisms**
- **Failover Strategies:** Redundant processing pipelines ensure grading continuity.
- **Fallbacks for Diagram Processing:**
  - **High confidence** → Normal multimodal processing.
  - **Low confidence** → LLaVA-generated textual descriptions.
  - **Very low confidence** → OCR-based parsing.
  - **Complete failure** → Human-in-the-loop escalation.

## **7. Security & Compliance**

```mermaid
graph TD;
    A[User Authentication] -->|Login Request| B[OAuth 2.0 Authenticator]
    B -->|Validated Token| C[Role-Based Access Control]
    C -->|Access Granted| D[Grading System]
    D -->|Process Request| E[Audit Logging System]
    E -->|Store Logs| F[Compliance Storage]
    F -->|Review Logs| G[Security Monitoring System]
```

### **7.1 Authentication & Authorization**
- **OAuth 2.0 for API authentication**
- **Role-based access control (RBAC) for data security**

### **7.2 Compliance Considerations**
- **Data privacy standards** (GDPR, CCPA) enforced via access policies.
- **Audit logs** maintained for grading transparency.
### **6.1 Error Handling & Fallbacks**
- **Text Processing Failures:** Retry mechanism with fallback to simplified parsing.
- **Diagram Recognition Failures:**
  - **Low confidence → OCR-based extraction**
  - **Very low confidence → Human escalation**
- **Reference Checking Failures:** Use alternative search mechanisms.

### **6.2 Scalability Considerations**
- **Batch Processing:** Parallel grading pipelines for large-scale submissions.
- **Dynamic Scaling:** Auto-scaling based on submission volume.

---

## **7. Security & Compliance**
| **Security Measure** | **Implementation** |
|---------------------|------------------|
| **Authentication** | OAuth 2.0 for secure API calls |
| **Access Control** | RBAC to restrict grading data access |
| **Data Privacy** | Compliance with GDPR & CCPA |

---

## **8. Conclusion**
This Level 2 architecture defines the **developer-focused orchestration** of the grading system, ensuring **scalability, fault tolerance, and explainability** while leveraging **GPT-4** for all processing tasks.

