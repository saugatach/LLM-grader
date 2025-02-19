# Architecture Decision Record (ADR) - Data Retention Strategy

### 1. Title
Data Retention and Backup Strategy for Case Study Grading System

### 2. Context
Need to maintain grading data for:
- Academic records
- Grade disputes
- System recovery
- Audit purposes

### 3. Comparison of Approaches

| Approach | Purpose | How It Works | Best Use Cases | Limitations |
|----------|----------|--------------|----------------|-------------|
| **Keep Everything** | Full history | Store all data | Unlimited storage | Expensive, unnecessary |
| **Minimal Retention** | Current data only | Keep recent only | Simple systems | Risk of data loss |
| **Time-Based Policy (Selected)** | Balance retention | Keep data based on age | Most applications | Needs clear rules |

### 4. Decision
Selected Time-Based Policy with Tiered Storage:
- Active data in standard storage
- Archived data in cold storage
- Automated lifecycle policies
- Cost-optimized approach

### 5. Storage Tiers and Technologies

| Storage Tier | Technology Options | Access Time | Cost | Use Case |
|--------------|-------------------|-------------|------|-----------|
| Hot Storage | - AWS S3 Standard<br>- Azure Blob Storage<br>- Google Cloud Storage | Immediate | Higher | Current semester papers |
| Cool Storage | - AWS S3 IA<br>- Azure Cool Tier<br>- GCP Nearline | Hours | Medium | Last year papers |
| Cold Storage | - AWS Glacier<br>- Azure Archive<br>- GCP Coldline | Days | Lowest | Old papers, audit data |

### 6. Retention Rules

| Data Type | Retention Period | Storage Tier | Lifecycle |
|-----------|------------------|--------------|-----------|
| Current Papers | 6 months | Hot | Move to cool after semester |
| Recent Papers | 1 year | Cool | Move to cold after 1 year |
| Old Papers | 5 years | Cold | Delete after 5 years |
| Grades | 5 years | Cool | Never delete |
| System Logs | 3 months | Cool | Auto-delete |

### 7. Data Lifecycle Flows
Each data type follows a defined lifecycle path:
- **Papers**: Hot → Cool → Cold → Delete
- **Grades**: Hot → Cool → Archive
- **Logs**: Hot → Cool → Delete

Note: Detailed lifecycle diagrams are available in the Level 3 design document.

### 8. Consequences

#### Positive Impact
- Cost-effective storage
- Clear retention rules
- Automated lifecycle
- Meets compliance needs
- Simple to implement

#### Negative Impact
- Retrieval delays for old data
- Storage tier management
- Retrieval costs for cold data

### 9. Conclusion
Time-based retention with tiered storage provides cost-effective data management while maintaining accessibility when needed. The automated lifecycle management ensures data moves through appropriate storage tiers based on age and usage patterns. 