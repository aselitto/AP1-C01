# AWS AI Practitioner (AIP-C01) Mastery Guide
## Distilled from 900 Exam Questions

---

## 🎯 What This Exam REALLY Tests

You're right - this is **Solutions Architect thinking for GenAI**, not coding. The exam tests whether you can:
1. **Choose the right AWS service** for a GenAI scenario
2. **Design architectures** that balance cost, performance, security, and compliance
3. **Know when to use what** - not how to code it

---

## 📊 The 5 Core Competency Domains

### Domain 1: Implementation & Integration (30%)
**What you must know:**
- API patterns: `InvokeModel` vs `Converse` vs `ConverseStream`
- When to use Agents vs Knowledge Bases vs direct model calls
- Lambda + API Gateway + Bedrock patterns
- SageMaker endpoints: Real-time vs Serverless vs Async vs Batch Transform
- Multi-model endpoints vs Inference Components

### Domain 2: AI Safety, Security & Governance (25%)
**What you must know:**
- Guardrails: Content filters, Topic filters, PII filters (BLOCK vs ANONYMIZE)
- IAM condition keys: `bedrock:GuardrailIdentifier`
- Data residency: Cross-region inference, FIPS endpoints
- Model invocation logging to S3/CloudWatch
- Tokenization for PII recovery

### Domain 3: Data Management & Compliance (20%)
**What you must know:**
- Vector stores: OpenSearch vs Aurora pgvector vs S3 Vectors vs ElastiCache
- Chunking strategies: Fixed, Semantic, Hierarchical
- Embedding models: Titan Embeddings, Cohere Multilingual
- Data validation: AWS Glue Data Quality, DQDL rules

### Domain 4: Operational Efficiency & Cost Optimization (15%)
**What you must know:**
- Pricing tiers: On-demand vs Provisioned vs Batch (50% savings)
- Service tiers: Priority vs Standard vs Flex
- Prompt caching (90% savings on cached tokens)
- Model distillation (75% cost reduction)
- Intelligent prompt routing (30% savings)
- Cross-region inference for availability

### Domain 5: Testing, Validation & Troubleshooting (10%)
**What you must know:**
- Model evaluation: Programmatic vs LLM-as-Judge vs Human
- RAG evaluation: Faithfulness, Completeness, Context Relevance
- Troubleshooting: Throttling (exponential backoff + jitter)
- CloudWatch metrics: `InvocationsIntervened`, `GuardrailPolicyType`

---

## 🔑 The 50 Key Decisions You Must Master

### Bedrock API Selection
| Scenario | Use This |
|----------|----------|
| Simple single-turn | `InvokeModel` |
| Multi-turn conversation | `Converse` API (unified, model-agnostic) |
| Real-time streaming | `ConverseStream` or `InvokeModelWithResponseStream` |
| Large batch processing | Batch Inference (50% cost savings) |
| Model migration | Converse API (change modelId only) |

### Guardrails Configuration
| Need | Configuration |
|------|---------------|
| Block harmful content | Content filters with strength levels |
| Block specific topics | Denied topics with descriptions |
| Redact PII but keep context | `ANONYMIZE` mode |
| Block all PII | `BLOCK` mode |
| Recover original PII | Tokenization service + Lambda |
| Monitor interventions | CloudWatch `AWS/Bedrock/Guardrails` namespace |

### Cost Optimization
| Scenario | Solution |
|----------|----------|
| Non-real-time bulk processing | Batch inference (50% off) |
| Repeated context in prompts | Prompt caching (90% off cached tokens) |
| Variable complexity queries | Intelligent prompt routing (30% off) |
| Predictable high volume | Provisioned throughput (40-60% off) |
| Simple queries at scale | Model distillation (75% off) |
| Non-urgent workloads | Flex tier |
| Mission-critical real-time | Priority tier |

### Vector Store Selection
| Scenario | Use This |
|----------|----------|
| Sub-millisecond cache | ElastiCache for Redis |
| Graph relationships (GraphRAG) | Neptune Analytics |
| Cost-effective archival | S3 Vectors |
| Full-text + vector search | OpenSearch |
| Transactional + vector | Aurora PostgreSQL pgvector |
| Multilingual search | Titan Embeddings V2 or Cohere Multilingual |

### SageMaker Endpoint Selection
| Scenario | Use This |
|----------|----------|
| Consistent high traffic | Real-time endpoint |
| Variable traffic, can tolerate cold starts | Serverless inference |
| Large payloads, long processing | Async inference |
| Bulk offline processing | Batch transform |
| Multiple models, similar traffic | Multi-model endpoint |
| One model needs 100x traffic | Dedicated endpoint for that model |
| Scale to zero needed | Inference components |

### Error Handling
| Error | Solution |
|-------|----------|
| ThrottlingException | Exponential backoff WITH JITTER |
| ModelNotReadyException | Retry up to 10 times with backoff |
| Large response truncation | Check guardrail streaming config |
| Inconsistent RAG results | Re-index after embedding model change |

---

## 🏗️ TWO PROJECTS THAT COVER EVERYTHING

### Project 1: Enterprise Customer Service Platform
**Covers: 80% of exam topics**

Build a customer service chatbot that:

#### Phase 1: Core Implementation
- [ ] Use Bedrock `Converse` API for multi-turn conversations
- [ ] Store conversation history in DynamoDB
- [ ] Deploy via API Gateway + Lambda
- [ ] Implement streaming responses with WebSocket API

#### Phase 2: Knowledge Base (RAG)
- [ ] Create Knowledge Base with S3 data source
- [ ] Configure hierarchical chunking (parent 1500 tokens, child 300)
- [ ] Use OpenSearch Serverless as vector store
- [ ] Implement metadata filtering for document types
- [ ] Enable reranking for better relevance

#### Phase 3: Safety & Compliance
- [ ] Create Guardrails with:
  - Content filters (hate, violence, etc.)
  - Denied topics (competitor mentions, legal advice)
  - PII filters with ANONYMIZE mode
- [ ] Integrate tokenization service for PII recovery
- [ ] Enable guardrail tracing
- [ ] Monitor `InvocationsIntervened` by `GuardrailPolicyType`

#### Phase 4: Agent Capabilities
- [ ] Create Bedrock Agent with action groups
- [ ] Implement "return control" for async operations
- [ ] Add code interpretation for data analysis
- [ ] Configure knowledge base integration

#### Phase 5: Cost Optimization
- [ ] Implement prompt caching for system prompts
- [ ] Use intelligent prompt routing (Claude Sonnet ↔ Haiku)
- [ ] Route non-urgent requests to Flex tier
- [ ] Set up batch inference for nightly reports

#### Phase 6: Monitoring & Evaluation
- [ ] Enable model invocation logging to S3
- [ ] Create CloudWatch dashboard for:
  - Latency metrics
  - Token usage
  - Guardrail interventions
  - Error rates
- [ ] Set up LLM-as-Judge evaluation jobs
- [ ] Configure human evaluation workflow with SageMaker work teams

#### Phase 7: Multi-Region & Resiltic
- [ ] Configure cross-region inference profile
- [ ] Implement exponential backoff with jitter
- [ ] Set up DLQ for failed requests

---

### Project 2: Document Processing Pipeline
**Covers: Remaining 20% + reinforces core concepts**

Build a document analysis system that:

#### Phase 1: Data Ingestion
- [ ] Use Bedrock Data Automation for document extraction
- [ ] Configure custom blueprints for document types
- [ ] Implement AWS Glue Data Quality rules (DQDL)
- [ ] Route failed validations to manual review

#### Phase 2: Embedding & Storage
- [ ] Generate embeddings with Titan Embeddings V2
- [ ] Store in Aurora PostgreSQL with pgvector
- [ ] Implement semantic chunking
- [ ] Handle multilingual content

#### Phase 3: Batch Processing
- [ ] Configure batch inference jobs
- [ ] Schedule with EventBridge
- [ ] Store results in S3 with lifecycle policies
- [ ] Monitor with CloudWatch batch metrics

#### Phase 4: Model Customization
- [ ] Fine-tune model for domain-specific tasks
- [ ] Deploy with provisioned throughput
- [ ] Implement model versioning
- [ ] Set up A/B testing with shadow deployments

#### Phase 5: Compliance
- [ ] Enable S3 Object Lock for audit retention
- [ ] Configure FIPS endpoints
- [ ] Implement data residency controls
- [ ] Set up 10-year retention policies

---

## 📋 Quick Reference: Service Selection Matrix

```
┌─────────────────────────────────────────────────────────────────┐
│                    WHEN TO USE WHAT                              │
├─────────────────────────────────────────────────────────────────┤
│ REAL-TIME CHAT        → Converse API + Streaming                │
│ DOCUMENT Q&A          → Knowledge Bases + RAG                   │
│ COMPLEX WORKFLOWS     → Bedrock Agents                          │
│ BULK PROCESSING       → Batch Inference                         │
│ CUSTOM MODELS         → Fine-tuning + Provisioned Throughput    │
│ SAFETY FILTERS        → Guardrails                              │
│ COST OPTIMIZATION     → Prompt Caching + Intelligent Routing    │
│ MULTI-REGION          → Cross-Region Inference Profiles         │
├─────────────────────────────────────────────────────────────────┤
│ VECTOR STORE NEEDS                                               │
├─────────────────────────────────────────────────────────────────┤
│ Speed + Caching       → ElastiCache Redis                       │
│ Graph Relationships   → Neptune Analytics                       │
│ Cost-Effective        → S3 Vectors                              │
│ Full-Text Search      → OpenSearch                              │
│ Transactional         → Aurora pgvector                         │
├─────────────────────────────────────────────────────────────────┤
│ SAGEMAKER ENDPOINTS                                              │
├─────────────────────────────────────────────────────────────────┤
│ Consistent Traffic    → Real-time                               │
│ Spiky Traffic         → Serverless (no GPU)                     │
│ Large Payloads        → Async                                   │
│ Offline Bulk          → Batch Transform                         │
│ Many Models           → Multi-model                             │
│ Scale to Zero         → Inference Components                    │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🎓 The 10 "Gotchas" That Appear Repeatedly

1. **Converse API is model-agnostic** - Use it for easy model migration
2. **Batch inference = 50% savings** - Always consider for non-real-time
3. **Prompt caching expires in 5 minutes** - Not for unique content
4. **ANONYMIZE vs BLOCK** - ANONYMIZE keeps context, BLOCK stops everything
5. **Exponential backoff MUST have jitter** - Prevents thundering herd
6. **Embedding model changes require re-indexing** - Vectors are incompatible
7. **Lambda doesn't support GPU** - Use SageMaker for GPU inference
8. **Serverless inference doesn't support GPU** - Use real-time endpoints
9. **Cross-region inference has no extra routing cost** - Price based on source region
10. **Model invocation logs don't store prompts by default** - Must enable logging

---

## ✅ Pre-Exam Checklist

Before the exam, make sure you can answer:

### Implementation
- [ ] When would I use Converse vs InvokeModel?
- [ ] How do I maintain conversation context?
- [ ] When should I use an Agent vs Knowledge Base?
- [ ] How do I implement async processing with Agents?

### Security
- [ ] How do I enforce guardrails organization-wide?
- [ ] What's the difference between BLOCK and ANONYMIZE?
- [ ] How do I recover PII after anonymization?
- [ ] What IAM condition keys are available for Bedrock?

### Data
- [ ] Which vector store for which scenario?
- [ ] What chunking strategy for code? For conversations?
- [ ] How do I handle multilingual content?

### Cost
- [ ] What's the cost savings for each optimization?
- [ ] When is provisioned throughput better than on-demand?
- [ ] How does prompt caching work?

### Operations
- [ ] What CloudWatch metrics should I monitor?
- [ ] How do I evaluate RAG quality?
- [ ] What's the proper retry strategy for throttling?

---

## 🚀 Summary: Build These Two Apps

**If you build these two projects, you'll have hands-on experience with every major exam topic:**

1. **Customer Service Platform** - Covers conversational AI, RAG, Agents, Guardrails, cost optimization, monitoring
2. **Document Processing Pipeline** - Covers batch processing, data validation, embeddings, compliance, model customization

The exam is about **knowing which service to use when** - not about writing code. These projects give you the mental models to make those decisions confidently.

---

*Generated from analysis of 900 practice questions for AWS AI Practitioner (AIP-C01)*
