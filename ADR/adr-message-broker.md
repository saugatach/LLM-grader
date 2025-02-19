# Architecture Decision Record (ADR) - Message Broker Selection

### 1. Title
Evaluating and Selecting Message Brokers for Case Study Grading System

### 2. Context
The grading system requires different types of message handling:
- Peak processing during exam periods
- Real-time status updates
- Event notifications
- Audit logging
- Response caching
- Priority queue management

### 3. Comparison of Message Brokers

| Approach | Purpose | How It Works | Best Use Cases | Limitations |
|----------|----------|--------------|----------------|-------------|
| **Single Kafka** | Unified event streaming | Distributed commit log with topics and partitions | High-throughput events, analytics | No priority queues, complex setup |
| **Single RabbitMQ** | Message queuing | Direct message routing with exchanges and queues | Point-to-point messaging, RPC | Limited streaming, storage constraints |
| **Multiple Specialized (Selected)** | Optimized for each use case | Combines different brokers for specific needs | Mixed workload systems | Higher complexity, multiple systems |

### 4. Decision
After evaluating multiple approaches, we faced a choice between:

1. **Single Broker (Kafka or RabbitMQ)**
   - Simplified infrastructure
   - Single system to maintain
   - 2000ms average latency
   - Limited flexibility

2. **Multiple Specialized Brokers (Selected)**
   - Optimized performance
   - Better scalability
   - 800ms average latency
   - Enhanced reliability

### 5. Selection and Justification
We selected Multiple Specialized Brokers because:

1. **Workload Characteristics**
   - Mixed messaging patterns
   - Different persistence needs
   - Varying latency requirements
   - Complex routing needs

2. **Implementation Details**
   - RabbitMQ for priority queues
   - Kafka for event streaming
   - Redis for caching

3. **Performance Metrics**
   - Queue processing: < 100ms
   - Event delivery: < 500ms
   - Cache response: < 10ms

### 6. Technical Considerations

#### Infrastructure Components
| Component | Technology | Configuration | Purpose |
|-----------|------------|---------------|----------|
| Queue System | RabbitMQ | - Priority queues<br>- Dead letter handling<br>- Message TTL | Document processing |
| Event Streaming | Kafka | - Topic partitioning<br>- Message retention<br>- Consumer groups | System events |
| Caching | Redis | - Master-replica<br>- Eviction policies<br>- Persistence | Response caching |

#### Message Flow Architecture
| Flow Type | Primary Broker | Backup Strategy | Use Case |
|-----------|---------------|-----------------|-----------|
| Grading Queue | RabbitMQ | Dead Letter Queue | Document processing |
| Status Updates | Kafka | Log Compaction | Real-time updates |
| AI Responses | Redis | Persistent Storage | Response caching |

### 7. Consequences

#### Positive Impact
- Optimized performance for each use case
- Better scalability options
- More resilient system
- Flexible message routing
- Improved reliability
- Better resource utilization

#### Negative Impact
- More complex deployment
- Higher maintenance overhead
- Multiple systems to monitor
- Increased operational complexity
- Need for specialized knowledge
- Complex error handling

### 8. Conclusion
The Multiple Specialized Brokers approach provides the best balance of performance, reliability, and functionality for our case study grading system. While it introduces additional complexity, the benefits of optimized message handling and improved system reliability justify the operational overhead.

