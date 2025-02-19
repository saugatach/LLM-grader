# Architecture Decision Record (ADR) - Monitoring Strategy

### 1. Title
OpenTelemetry with Grafana for Case Study Grading System

### 2. Context
Need simple monitoring to track system health and performance.

### 3. Comparison of Approaches

| Approach | Purpose | How It Works | Best Use Cases | Limitations |
|----------|----------|--------------|----------------|-------------|
| **Logs Only** | Basic tracking | File logs | Simple apps | Hard to visualize |
| **Multiple Tools** | Full monitoring | Many systems | Large enterprises | Too complex |
| **OpenTelemetry + Grafana (Selected)** | Essential monitoring | Single collection, simple display | Modern apps | Learning curve |

### 4. Decision
Selected OpenTelemetry with Grafana because:
- OpenTelemetry collects all data
- Grafana displays it simply
- No extra tools needed
- Industry standard
- Easy to use

### 5. Dashboard Views

1. **Main Dashboard**
   - System status
   - Error rates
   - Papers processed
   - Processing times

2. **Resource Dashboard**
   - Memory usage
   - CPU load
   - Storage space
   - Queue length

### 6. Consequences

#### Positive Impact
- Simple setup
- Clear visibility
- Easy maintenance
- Standard tools

#### Negative Impact
- Basic metrics only
- Learning curve

### 7. Conclusion
OpenTelemetry collecting data and Grafana showing it provides all the monitoring we need without complexity. 