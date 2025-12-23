# AIP-C01 Decision Flowcharts

Generated from 900 practice questions. Use these diagrams to quickly identify the right AWS service for common GenAI scenarios.

## Table of Contents
1. [Quick Decision Matrix](#quick-decision-matrix)
2. [Vector Store Selection](#vector-store-selection)
3. [Endpoint Selection](#endpoint-selection)
4. [PII Handling](#pii-handling)
5. [Cost Optimization](#cost-optimization)
6. [Service Relationships](#service-relationships)
7. [Domain Coverage](#domain-coverage)

---

## Quick Decision Matrix

The most important decision patterns distilled from all questions:

```mermaid
flowchart TB
    subgraph "🎯 Quick Decision Matrix"
        direction TB
        
        subgraph "Latency Requirements"
            L1["< 100ms → Real-time endpoint + Priority tier"]
            L2["< 500ms → Standard tier or Provisioned"]
            L3["Seconds OK → Flex tier or Serverless"]
            L4["No SLA → Batch inference (50% savings)"]
        end
        
        subgraph "Scale Requirements"
            S1["Variable traffic → Auto-scaling + Scale-to-zero"]
            S2["Consistent high → Provisioned Throughput"]
            S3["Spiky → SQS buffer + Lambda"]
            S4["GPU needed → SageMaker Real-time (not Lambda!)"]
        end
        
        subgraph "Data Requirements"
            D1["PII + Recovery → ANONYMIZE + Tokenization"]
            D2["PII + Block → BLOCK mode"]
            D3["Embedding change → Must re-index!"]
            D4["Multi-region → Cross-region inference"]
        end
        
        subgraph "Cost Optimization"
            C1["Repeated prompts → Prompt caching (90% off)"]
            C2["Batch OK → Batch inference (50% off)"]
            C3["Large vectors → Titan 256-dim (75% less)"]
            C4["Archive data → S3 Vectors (90% less)"]
        end
    end
    
    style L1 fill:#FFE4E1
    style L2 fill:#FFE4E1
    style L3 fill:#FFE4E1
    style L4 fill:#FFE4E1
    style S1 fill:#E0FFE0
    style S2 fill:#E0FFE0
    style S3 fill:#E0FFE0
    style S4 fill:#E0FFE0
    style D1 fill:#E0E0FF
    style D2 fill:#E0E0FF
    style D3 fill:#E0E0FF
    style D4 fill:#E0E0FF
    style C1 fill:#FFFFD0
    style C2 fill:#FFFFD0
    style C3 fill:#FFFFD0
    style C4 fill:#FFFFD0
```

---

## Vector Store Selection

When to use which vector database:

```mermaid
flowchart TD
    START[Vector Store Selection] --> Q1{Data Volume?}
    
    Q1 -->|100M+ documents| Q2{Update Frequency?}
    Q1 -->|< 100M documents| Q3{Latency Requirement?}
    
    Q2 -->|Monthly/Batch| S3V[S3 Vectors<br/>90% cost reduction<br/>subsecond queries]
    Q2 -->|Real-time| OSS[OpenSearch Serverless<br/>Auto-scaling<br/>No infra mgmt]
    
    Q3 -->|Sub-millisecond| EC[ElastiCache<br/>In-memory vectors<br/>Real-time updates]
    Q3 -->|< 100ms| Q4{Need SQL/ACID?}
    
    Q4 -->|Yes| APG[Aurora PostgreSQL<br/>pgvector extension<br/>Transactional]
    Q4 -->|No| Q5{Graph Relationships?}
    
    Q5 -->|Yes| NEP[Neptune Analytics<br/>Graph + Vector<br/>Entity relationships]
    Q5 -->|No| OSS
    
    style S3V fill:#90EE90
    style OSS fill:#87CEEB
    style EC fill:#FFB6C1
    style APG fill:#DDA0DD
    style NEP fill:#F0E68C
```

### Key Takeaways:
- **S3 Vectors**: Best for large archives with infrequent updates (90% cost reduction)
- **OpenSearch Serverless**: Best for auto-scaling without infrastructure management
- **ElastiCache**: Best for sub-millisecond latency with real-time updates
- **Aurora PostgreSQL**: Best when you need ACID transactions + vector search
- **Neptune Analytics**: Best when you need graph relationships + vector search

---

## Endpoint Selection

Choosing the right inference endpoint:

```mermaid
flowchart TD
    START[Inference Endpoint Selection] --> Q1{GPU Required?}
    
    Q1 -->|Yes| Q2{Traffic Pattern?}
    Q1 -->|No| Q3{Latency Requirement?}
    
    Q2 -->|Variable/Spiky| SMRT[SageMaker Real-time<br/>+ Scale-to-Zero<br/>GPU + Auto-scaling]
    Q2 -->|Consistent High| PT[Bedrock Provisioned<br/>Throughput<br/>Dedicated capacity]
    Q2 -->|Batch/Overnight| BATCH[Bedrock Batch Inference<br/>50% cost savings<br/>No latency SLA]
    
    Q3 -->|< 500ms critical| Q4{Traffic Volume?}
    Q3 -->|Seconds OK| Q5{Payload Size?}
    
    Q4 -->|High sustained| PT
    Q4 -->|Variable| TIER[Bedrock Service Tiers<br/>Priority/Standard/Flex<br/>Match workload to tier]
    
    Q5 -->|< 6MB| SMSL[SageMaker Serverless<br/>No GPU<br/>Scale to zero]
    Q5 -->|> 6MB or long| SMAS[SageMaker Async<br/>Large payloads<br/>Long processing]
    
    style SMRT fill:#90EE90
    style PT fill:#FFD700
    style BATCH fill:#87CEEB
    style TIER fill:#DDA0DD
    style SMSL fill:#F0E68C
    style SMAS fill:#FFB6C1
```

### Key Takeaways:
- **GPU Required?** → Only SageMaker Real-time supports GPU (NOT Lambda, NOT Serverless)
- **Variable Traffic?** → Use scale-to-zero feature (new in 2024!)
- **Batch OK?** → Always use Batch Inference for 50% savings
- **Mixed Workloads?** → Use Service Tiers (Priority/Standard/Flex)

---

## PII Handling

Guardrails configuration for sensitive data:

```mermaid
flowchart TD
    START[PII Handling Strategy] --> Q1{Need to Recover PII?}
    
    Q1 -->|Yes| TOK[Guardrails ANONYMIZE<br/>+ Tokenization Service<br/>Store PII mappings]
    Q1 -->|No| Q2{Context Important?}
    
    Q2 -->|Yes, keep conversation| ANON[Guardrails ANONYMIZE<br/>Replace with [EMAIL]<br/>Conversation continues]
    Q2 -->|No, block entirely| BLOCK[Guardrails BLOCK<br/>Stop request<br/>Return error message]
    
    TOK --> TRACK[Enable Guardrail Tracing<br/>CloudWatch InvocationsIntervened<br/>GuardrailPolicyType dimension]
    ANON --> TRACK
    BLOCK --> TRACK
    
    subgraph "Anti-Patterns"
        AP1[❌ Post-process with Comprehend<br/>PII already logged]
        AP2[❌ Client-side redaction<br/>Security risk]
        AP3[❌ Regex confidence scores<br/>Guardrails don't return scores]
    end
    
    style TOK fill:#90EE90
    style ANON fill:#87CEEB
    style BLOCK fill:#FFB6C1
    style TRACK fill:#DDA0DD
    style AP1 fill:#FF6B6B
    style AP2 fill:#FF6B6B
    style AP3 fill:#FF6B6B
```

### Key Takeaways:
- **ANONYMIZE (Mask)**: Replaces PII with placeholders, conversation continues
- **BLOCK**: Stops entire request, returns error
- **Tokenization**: Use with ANONYMIZE when you need to recover original PII
- **Never**: Post-process with Comprehend (PII already logged!) or client-side redaction

---

## Cost Optimization

Reducing GenAI costs:

```mermaid
flowchart TD
    START[Cost Optimization Strategy] --> Q1{Latency Requirement?}
    
    Q1 -->|No strict SLA| BATCH[Batch Inference<br/>50% cost reduction<br/>Process overnight]
    Q1 -->|Real-time needed| Q2{Traffic Pattern?}
    
    Q2 -->|Predictable high| Q3{Commitment OK?}
    Q2 -->|Variable/Spiky| Q4{Repeated Prompts?}
    Q2 -->|Mixed workloads| TIER[Service Tiers<br/>Priority: critical<br/>Standard: normal<br/>Flex: tolerant]
    
    Q3 -->|Yes, 1-6 months| PROV[Provisioned Throughput<br/>Committed discount<br/>Guaranteed capacity]
    Q3 -->|No commitment| OD[On-Demand<br/>Pay per token<br/>No commitment]
    
    Q4 -->|Yes, common prefixes| CACHE[Prompt Caching<br/>90% discount cached<br/>85% latency reduction]
    Q4 -->|No, unique content| Q5{Can use smaller model?}
    
    Q5 -->|Same quality needed| OD
    Q5 -->|Quality flexible| DIST[Model Distillation<br/>Train smaller model<br/>Reduce inference cost]
    
    subgraph "Vector Store Cost"
        VS1[S3 Vectors: 90% cheaper storage]
        VS2[Titan 256-dim: 75% less storage]
        VS3[OpenSearch Serverless: Pay per use]
    end
    
    style BATCH fill:#90EE90
    style PROV fill:#FFD700
    style CACHE fill:#87CEEB
    style TIER fill:#DDA0DD
    style DIST fill:#F0E68C
    style VS1 fill:#98FB98
    style VS2 fill:#98FB98
    style VS3 fill:#98FB98
```

### Key Takeaways:
- **Batch Inference**: 50% cost reduction for non-real-time workloads
- **Prompt Caching**: 90% discount on cached tokens, 85% latency improvement
- **Service Tiers**: Match workload urgency to tier (Priority > Standard > Flex)
- **Vector Dimensions**: Titan 256-dim maintains 97% accuracy with 75% less storage

---

## Service Relationships

How AWS services connect in GenAI architectures:

```mermaid
flowchart LR
    subgraph "Core GenAI Services"
        BR[Amazon Bedrock<br/>376 questions]
        SM[SageMaker<br/>48 questions]
    end
    
    subgraph "Compute & Integration"
        LAM[Lambda<br/>162 questions]
        SF[Step Functions<br/>32 questions]
        AG[API Gateway<br/>32 questions]
    end
    
    subgraph "Data & Storage"
        S3[S3<br/>50 questions]
        DDB[DynamoDB<br/>43 questions]
        OS[OpenSearch<br/>27 questions]
    end
    
    subgraph "Monitoring & Security"
        CW[CloudWatch<br/>126 questions]
        IAM[IAM<br/>48 questions]
        KMS[KMS<br/>23 questions]
    end
    
    subgraph "Event & Streaming"
        EB[EventBridge<br/>63 questions]
        KIN[Kinesis<br/>39 questions]
        SQS[SQS<br/>19 questions]
    end
    
    BR --> LAM
    BR --> CW
    BR --> S3
    BR --> OS
    LAM --> DDB
    LAM --> SF
    AG --> LAM
    EB --> LAM
    KIN --> LAM
    SM --> S3
    SM --> CW
    IAM --> BR
    IAM --> SM
    KMS --> S3
    KMS --> BR
    
    style BR fill:#FF9900
    style SM fill:#FF9900
    style LAM fill:#FF9900
    style CW fill:#FF9900
```

---

## Domain Coverage

Question distribution across exam domains:

```mermaid
pie showData
    title Question Distribution by Domain
    "Foundation Model & Data (31%)" : 273
    "Implementation & Integration (26%)" : 233
    "AI Safety & Governance (20%)" : 156
    "Operational Efficiency (14%)" : 126
    "Testing & Troubleshooting (12%)" : 106
```

---

## Anti-Patterns to Avoid

Common wrong answers from the questions:

| Scenario | Wrong Choice | Why It's Wrong |
|----------|--------------|----------------|
| GPU inference | Lambda | Lambda doesn't support GPU |
| PII logging | Comprehend post-process | PII already logged before detection |
| Async pattern | Step Functions with Agent | Agents can't integrate with Step Functions |
| Embedding change | Keep old vectors | Must re-index when embedding model changes |
| High latency tolerance | Priority tier | Wastes money - use Flex tier |
| Variable traffic | Fixed provisioned | Use auto-scaling or scale-to-zero |

---

*Generated from 900 AIP-C01 practice questions*
