# AWS Certified AI Practitioner (AP1-C01) Study Kit

A comprehensive study resource for the **AWS Certified Generative AI Developer - Professional (AP1-C01)** exam, featuring 900 practice questions, decision flowcharts, and a hands-on RAG Training Chatbot you can build yourself.

## 🎯 What's Included

| Resource | Description |
|----------|-------------|
| **900 Practice Questions** | Parsed and structured questions covering all 5 exam domains |
| **Decision Flowcharts** | Visual guides for choosing the right AWS service |
| **RAG Training Chatbot** | Build your own study assistant using Amazon Bedrock |
| **Mastery Guide** | Distilled knowledge from all questions |

## 📊 Exam Domain Coverage

Based on analysis of 900 practice questions:

| Domain | Weight | Questions |
|--------|--------|-----------|
| Foundation Model Integration, Data Management & Compliance | 31% | 273 |
| Implementation and Integration | 26% | 233 |
| AI Safety, Security, and Governance | 20% | 156 |
| Operational Efficiency and Optimization | 14% | 126 |
| Testing, Validation, and Troubleshooting | 12% | 106 |

## 🚀 Quick Start

### Option 1: Study with the Materials
1. Read `MASTERY-GUIDE.md` for key concepts
2. Review `decision-charts/DECISION-FLOWCHARTS.md` for visual decision trees
3. Use `parsed_questions.json` with the RAG chatbot

### Option 2: Build the RAG Training Chatbot
Follow the step-by-step guide in `BUILD-GUIDE.md` to create your own AI-powered study assistant using:
- Amazon Bedrock Knowledge Bases
- Amazon Bedrock Guardrails
- Claude 3 (or your preferred model)

## 📁 Project Structure

```
aws-ap1-c01/
├── README.md                    # This file
├── BUILD-GUIDE.md               # Step-by-step chatbot build instructions
├── MASTERY-GUIDE.md             # Key concepts distilled from all questions
├── parsed_questions.json        # Structured question data for RAG (upload to S3)
├── fundamentals.md              # GenAI fundamentals primer
│
├── exam-guide/                  # Official AWS exam documentation
│   ├── cd1.md - cd5.md          # Exam domain details
│   ├── Intro.md                 # Exam overview
│   └── others.md                # Additional topics
│
├── decision-charts/             # Visual decision flowcharts
│   ├── DECISION-FLOWCHARTS.md   # Mermaid diagrams (source)
│   └── *.png                    # Exported flowchart images
│
└── archive/                     # Raw questions & processing scripts
```

## 🔑 Key Decision Patterns

### When to Use What (Quick Reference)

**Latency Requirements:**
- `< 100ms` → Real-time endpoint + Priority tier
- `< 500ms` → Standard tier or Provisioned Throughput
- `Seconds OK` → Flex tier or Serverless
- `No SLA` → Batch inference (50% cost savings)

**PII Handling:**
- Need to recover PII later → ANONYMIZE + Tokenization
- Block entirely → BLOCK mode
- Never: Post-process with Comprehend (PII already logged!)

**Vector Store Selection:**
- 100M+ docs, monthly updates → S3 Vectors (90% cheaper)
- Auto-scaling needed → OpenSearch Serverless
- Sub-ms latency → ElastiCache
- Need ACID/SQL → Aurora PostgreSQL + pgvector

**Cost Optimization:**
- Repeated prompts → Prompt caching (90% discount)
- Batch workloads → Batch inference (50% off)
- Large vectors → Titan 256-dim (75% less storage)

## 🛠️ Building the RAG Chatbot

The chatbot uses these AWS services (all covered on the exam!):

| Service | Purpose | Exam Domain |
|---------|---------|-------------|
| Bedrock Knowledge Base | RAG retrieval | Domain 1 |
| Bedrock Guardrails | Content safety | Domain 3 |
| S3 | Document storage | Domain 1 |
| CloudWatch | Monitoring | Domain 4 |
| Claude 3 / Titan | LLM inference | Domain 2 |

See `BUILD-GUIDE.md` for complete instructions.

## 📈 Top Services to Know

From analyzing 900 questions, these services appear most frequently:

1. **Amazon Bedrock** - 376 questions (42%)
2. **AWS Lambda** - 162 questions (18%)
3. **Amazon CloudWatch** - 126 questions (14%)
4. **Amazon EventBridge** - 63 questions (7%)
5. **Amazon S3** - 50 questions (6%)

## ⚠️ Common Anti-Patterns

Things the exam tests you on NOT doing:

| Scenario | Wrong Choice | Why |
|----------|--------------|-----|
| GPU inference | Lambda | Lambda doesn't support GPU |
| PII protection | Comprehend post-process | PII already logged before detection |
| Async agent pattern | Step Functions | Agents can't integrate with Step Functions directly |
| Embedding model change | Keep old vectors | Must re-index when embedding model changes |
| Variable traffic | Fixed provisioned capacity | Use auto-scaling or scale-to-zero |

## 🤝 Contributing

1. Fork this repository
2. Add more practice questions or improve explanations
3. Submit a pull request

## 📚 Additional Resources

- [AWS Exam Guide](https://aws.amazon.com/certification/certified-generative-ai-developer-professional/)
- [Amazon Bedrock Documentation](https://docs.aws.amazon.com/bedrock/)
- [Tutorials Dojo Practice Exams](https://tutorialsdojo.com/)

## 📝 License

This study material is for educational purposes. Practice questions are derived from publicly available exam preparation resources.

---

*Good luck on your AIP-C01 exam! 🎓*
