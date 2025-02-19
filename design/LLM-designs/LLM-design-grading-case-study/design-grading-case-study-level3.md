# Case Study Grading System - Level 3 Technical Design

## **1. API Documentation**
Complete API specifications are maintained in OpenAPI format under `/design/api/`:
- Main specification: `openapi.yaml`
- Path definitions: `paths/*.yaml`
- Data schemas: `schemas/*.yaml`

### **1.2 Authentication & Authorization**
- JWT-based authentication
- Token format and validation rules
- Role-based access control implementation

### **1.3 Error Handling**
- Standardized error responses
- Error logging and monitoring
- Retry mechanisms for transient failures

### **1.4 Rate Limiting**
- Per-endpoint limits
- Burst allowances
- Client identification strategy

## **2. Image Processing Implementation**

### **2.1 Image Verification Pipeline**
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
```

### **2.2 Processing Thresholds**
```yaml
Confidence:
  HighConfidence: 0.85
  MediumConfidence: 0.60
  LowConfidence: 0.30

ImageConstraints:
  MaxSize: 10MB
  SupportedFormats: [PNG, JPG, SVG]
  Resolution:
    Min: 300x300
    Max: 4096x4096

ProcessingPaths:
  High:
    Service: GPT-4V
    Timeout: 30s
  Medium:
    Service: OCR
    Engine: Tesseract
  Low:
    Service: LLaVA
    Model: v1.5
  VeryLow:
    Service: HITL
    MaxWaitTime: 24h
```

## **3. Core System Implementation**

### **3.1 Document Processing Service Architecture**

```mermaid
sequenceDiagram
    participant Client
    participant LoadBalancer
    participant Processor1
    participant Processor2
    participant Storage
    
    Client->>LoadBalancer: Upload Document
    LoadBalancer->>Storage: Store Original
    
    par Parallel Processing
        LoadBalancer->>Processor1: Stream Chunk 1
        LoadBalancer->>Processor2: Stream Chunk 2
    end
    
    Processor1-->>Client: Processing Status 1
    Processor2-->>Client: Processing Status 2
    
    par Store Results
        Processor1->>Storage: Store Result 1
        Processor2->>Storage: Store Result 2
    end
```

#### Technical Architecture
- **Protocol**: gRPC
- **Load Balancing**: Round-robin with health checks
- **Streaming**: Bi-directional for real-time updates
- **Chunking**: 1MB optimal chunks

#### Processing Flow
1. Document received via gRPC stream
2. Split into optimal chunks
3. Parallel processing across nodes
4. Results aggregation
5. Status updates to client

### **3.2 Grading Pipeline Architecture**

```mermaid
flowchart TD
    subgraph Input["Input Processing"]
        A[Submission API] -->|New Document| B[Validator]
        B -->|Valid| C[Priority Router]
        B -->|Invalid| D[Error Handler]
    end
    
    subgraph Queues["Queue System"]
        C -->|Urgent| E[High Priority Queue]
        C -->|Normal| F[Standard Queue]
        
        E --> G[Grader Pool]
        F --> G
    end
    
    subgraph Processing["Grade Processing"]
        G -->|Chunks| H[Text Grader]
        G -->|Chunks| I[Image Grader]
        G -->|Chunks| J[Reference Grader]
        
        H & I & J -->|Scores| K[Aggregator]
    end
    
    subgraph Output["Result Handling"]
        K -->|Final Score| L[Result Queue]
        L -->|Notify| M[Event Bus]
        L -->|Store| N[Database]
    end
```

#### RabbitMQ Implementation
```yaml
Exchanges:
  grading:
    type: "topic"
    queues:
      high_priority:
        binding: "grade.urgent.*"
        arguments:
          x-max-priority: 10
          x-message-ttl: 3600000  # 1 hour
      standard:
        binding: "grade.normal.*"
        arguments:
          x-max-priority: 5
          x-message-ttl: 86400000  # 24 hours

  results:
    type: "direct"
    queues:
      aggregation:
        durable: true
        arguments:
          x-message-ttl: 86400000
      failed:
        durable: true
        arguments:
          x-dead-letter-exchange: "grading"
```

### **3.3 AI Service Integration**
```mermaid
sequenceDiagram
    participant GradingService
    participant RateLimit
    participant Cache
    participant GPT4
    participant ErrorHandler
    
    GradingService->>RateLimit: Check Limits
    alt Under Limit
        RateLimit-->>GradingService: Allowed
        GradingService->>Cache: Check Cache
        alt Cache Hit
            Cache-->>GradingService: Return Cached
        else Cache Miss
            GradingService->>GPT4: API Request
            GPT4-->>GradingService: Response
            GradingService->>Cache: Store
        end
    else Over Limit
        RateLimit-->>ErrorHandler: Throttled
        ErrorHandler-->>GradingService: Queue Request
    end
```

#### GPT-4 Integration Configuration
```yaml
APIConfig:
  endpoints:
    text_analysis: "/v1/completions"
    image_analysis: "/v1/images/analysis"
  
  rateLimit:
    requests_per_minute: 200
    burst: 250
    
  retry:
    max_attempts: 3
    backoff:
      initial: 1000  # ms
      multiplier: 2
      max: 8000  # ms

  cache:
    implementation: "Redis"
    ttl: 86400  # 24 hours
    maxSize: "10GB"
```

### **3.4 Event Notification System**
```mermaid
flowchart TD
    subgraph Publishers
        A[Grading Service]
        B[Document Processor]
        C[Review System]
    end
    
    subgraph Kafka["Kafka Cluster"]
        D[Topic: grading.events]
        E[Topic: system.metrics]
        F[Topic: audit.logs]
    end
    
    subgraph Consumers
        G[UI Updates]
        H[Notifications]
        I[Audit System]
    end
    
    A & B & C --> D & E & F
    D --> G & H
    F --> I
```

#### Kafka Configuration
```yaml
Topics:
  grading.events:
    partitions: 10
    replication_factor: 3
    retention:
      time: 604800000  # 7 days
      size: "10GB"
    
  system.metrics:
    partitions: 5
    replication_factor: 3
    retention:
      time: 172800000  # 2 days

ConsumerGroups:
  ui_updates:
    instances: 3
    config:
      auto.offset.reset: "latest"
      
  notifications:
    instances: 2
    config:
      auto.offset.reset: "earliest"
```

## **4. External Integration Architecture**

### **4.1 Storage Service**
```mermaid
flowchart TD
    subgraph Input["Document Input"]
        A[Upload Handler] -->|Large File| B[Multipart Upload]
        A -->|Small File| C[Direct Upload]
    end
    
    subgraph S3["S3 Storage"]
        B & C --> D[Raw Documents]
        E[Processing Results]
        F[Grading Artifacts]
    end
    
    D --> G[Document Processor]
    E --> H[Grading Service]
```

#### S3 Configuration
```yaml
Buckets:
  raw_documents:
    versioning: enabled
    lifecycle:
      transition_to_ia: 30  # days
      transition_to_glacier: 90  # days
    encryption: AES256
    
  processed_results:
    versioning: enabled
    lifecycle:
      expiration: 365  # days
    encryption: AES256

Upload:
  multipart_threshold: 10485760  # 10MB
  part_size: 5242880  # 5MB
  max_concurrency: 10
```

### **4.2 Human Review Interface**
```mermaid
flowchart TD
    subgraph Triggers
        A[Low Confidence] -->|< 70%| D[Review Queue]
        B[Manual Request] --> D
        C[Quality Check] --> D
    end
    
    subgraph Review["Review System"]
        D --> E[Reviewer Assignment]
        E --> F[Review Interface]
        F --> G[Decision]
    end
    
    G -->|Webhook| H[Grade Update]
    G -->|Event| I[Notification]
```

#### Webhook Configuration
```yaml
Endpoints:
  grade_review:
    url: "/api/v1/review"
    method: "POST"
    retry:
      max_attempts: 3
      backoff: exponential
    
  quality_check:
    url: "/api/v1/quality"
    method: "POST"
    timeout: 30000  # 30 seconds

Triggers:
  confidence_threshold: 0.70
  time_threshold: 300  # 5 minutes
  deviation_threshold: 0.20  # 20% from mean
```

### **4.3 External UI**
```mermaid
flowchart TD
    subgraph Client["Client Layer"]
        A[Web UI]
        B[Mobile UI]
    end
    
    subgraph API["API Gateway"]
        C[Upload Handler]
        D[Status Handler]
        E[Results Handler]
    end
    
    A & B -->|REST| C & D & E
    C --> F[Document Service]
    D & E --> G[Grading Service]
```

## **5. System Characteristics**

### **5.1 Performance Specifications**

#### Processing Capacity
```mermaid
graph TD
    A[System Capacity] --> B[Document Processing]
    A --> C[Grading Pipeline]
    A --> D[Storage]
    A --> E[AI Services]

    B --> B1[100 concurrent uploads]
    B --> B2[50MB/s throughput]
    
    C --> C1[1000 grades/hour]
    C --> C2[5ms queue latency]
    
    D --> D1[10TB total storage]
    D --> D2[1TB monthly growth]
    
    E --> E1[200 API calls/minute]
    E --> E2[2s response time]
```

#### Peak Load Handling
- Normal Load: 100 submissions/hour
- Peak Load: 500 submissions/hour (exam periods)
- Burst Capacity: 1000 submissions/hour (5 minutes max)
- Auto-scaling trigger: 70% resource utilization

### **5.2 Failure Recovery**

#### Component Failure Scenarios
```mermaid
stateDiagram-v2
    [*] --> Normal
    Normal --> Degraded: Service Issue
    Degraded --> Recovery: Auto-recover
    Recovery --> Normal: Success
    Degraded --> Manual: Auto-recovery Failed
    Manual --> Recovery: Human Intervention
```

#### Recovery Strategies
1. **AI Service Unavailable**
   - Fallback to cached responses (24h validity)
   - Queue new requests for retry
   - Alert if queue depth > 1000

2. **Queue System Overload**
   - Trigger consumer auto-scaling
   - Apply back pressure to submissions
   - Buffer in local storage

3. **Storage System Issues**
   - Use local caching (5GB buffer)
   - Async replication catch-up
   - Redirect to backup region

### **5.3 Monitoring Thresholds**

#### Key Metrics
```yaml
Processing:
  upload_latency_p95: 2s
  processing_time_p95: 30s
  error_rate_threshold: 1%

Grading:
  completion_time_p95: 5min
  accuracy_threshold: 95%
  confidence_score_min: 0.7

Storage:
  read_latency_p95: 100ms
  write_latency_p95: 200ms
  availability_target: 99.99%

AI Service:
  response_time_p95: 2s
  cache_hit_ratio_min: 40%
  error_rate_max: 0.5%
```

### **5.4 System Resilience**

#### Failure Recovery Strategies
```mermaid
stateDiagram-v2
    [*] --> Normal
    Normal --> Degraded: Service Issue
    Degraded --> Recovery: Auto-recover
    Recovery --> Normal: Success
    Degraded --> Manual: Auto-recovery Failed
    Manual --> Recovery: Human Intervention
```

#### Recovery Procedures
1. **AI Service Disruption**
   - Fallback to cached responses (24h validity)
   - Queue new requests for retry
   - Alert if queue depth > 1000

2. **Queue System Overload**
   - Trigger consumer auto-scaling
   - Apply back pressure to submissions
   - Buffer in local storage

3. **Storage System Issues**
   - Use local caching (5GB buffer)
   - Async replication catch-up
   - Redirect to backup region

### **5.5 Scaling Parameters**

#### Auto-scaling Rules
```yaml
DocumentProcessor:
  min_instances: 3
  max_instances: 10
  scale_up:
    cpu_threshold: 70%
    queue_depth: 100
  scale_down:
    cpu_threshold: 30%
    queue_depth: 10

GradingService:
  min_instances: 5
  max_instances: 15
  scale_up:
    grading_latency: 1min
    queue_depth: 500
  scale_down:
    idle_time: 10min
    queue_depth: 50
```

### Message Broker Performance Metrics

#### RabbitMQ Performance
```mermaid
graph TD
    subgraph Performance["Queue Performance Metrics"]
        A[Message Throughput] --> B[5000 msgs/sec]
        C[Latency] --> D[< 100ms p95]
        E[Queue Depth] --> F[Max 10000]
        G[Processing Time] --> H[< 200ms p95]
    end
```

| Metric | Target | Alert Threshold | Critical Threshold |
|--------|--------|----------------|-------------------|
| Queue Latency | < 100ms | > 200ms | > 500ms |
| Message Rate | 5000/sec | < 3000/sec | < 1000/sec |
| Dead Letter Rate | < 0.1% | > 0.5% | > 1% |
| Consumer Lag | < 100 | > 500 | > 1000 |

#### Kafka Performance
| Metric | Target | Alert Threshold | Critical Threshold |
|--------|--------|----------------|-------------------|
| Producer Latency | < 10ms | > 50ms | > 100ms |
| Consumer Lag | < 1000 msgs | > 5000 msgs | > 10000 msgs |
| Topic Throughput | 10000/sec | < 7000/sec | < 3000/sec |
| Replication Lag | < 100ms | > 500ms | > 1000ms |

#### Redis Cache Performance
| Metric | Target | Alert Threshold | Critical Threshold |
|--------|--------|----------------|-------------------|
| Response Time | < 10ms | > 50ms | > 100ms |
| Hit Rate | > 80% | < 60% | < 40% |
| Memory Usage | < 70% | > 80% | > 90% |
| Eviction Rate | < 100/sec | > 500/sec | > 1000/sec |

#### System-wide Message Processing
```yaml
Processing:
  grading_queue:
    peak_throughput: "5000/second"
    average_latency: "< 100ms"
    error_rate: "< 0.1%"
    recovery_time: "< 30s"

  event_streaming:
    throughput: "10000/second"
    end_to_end_latency: "< 500ms"
    data_loss: "zero"
    partition_count: 10

  caching:
    hit_ratio: "> 80%"
    eviction_rate: "< 100/second"
    replication_lag: "< 50ms"
```