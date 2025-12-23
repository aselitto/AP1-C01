# GenAI Fundamentals: From GPT-2 to AWS Bedrock

This document connects your existing GPT-2 knowledge to the AWS GenAI ecosystem and exam concepts.

---

## The Big Picture: How It All Connects

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        YOUR APPLICATION                                      │
│  (Web app, chatbot, document processor, code assistant, etc.)               │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                     AGENTIC AI LAYER (Orchestration)                        │
│  • Agents that plan, reason, and execute multi-step tasks                   │
│  • Like managers delegating to workers                                       │
│  • AWS: Bedrock Agents, Strands Agents, Step Functions                      │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                    ┌───────────────┼───────────────┐
                    ▼               ▼               ▼
            ┌───────────┐   ┌───────────┐   ┌───────────┐
            │   TOOLS   │   │    RAG    │   │  PROMPTS  │
            │           │   │           │   │           │
            │ Functions │   │ Knowledge │   │ Templates │
            │ APIs      │   │ Retrieval │   │ Context   │
            │ Databases │   │           │   │ Examples  │
            └───────────┘   └───────────┘   └───────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                     VECTOR STORE + EMBEDDINGS                               │
│  • Converts text → numbers (vectors) for semantic search                    │
│  • AWS: OpenSearch, Aurora pgvector, Bedrock Knowledge Bases                │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                     FOUNDATION MODELS (FMs)                                 │
│  • The "brain" - pre-trained LLMs like Claude, Llama, Titan                 │
│  • AWS: Amazon Bedrock (managed access to multiple FMs)                     │
│  • Similar to your GPT-2 but MUCH larger and more capable                   │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 1. Foundation Models (FMs) - The Brain

### What You Know (GPT-2)
You trained/fine-tuned GPT-2, which is a **transformer-based language model**:
- ~1.5 billion parameters
- Predicts next token based on context
- You controlled: tokenization, training data, hyperparameters

### What's Different in AWS/Production
**Foundation Models** are:
- MUCH larger (Claude 3 = ~200B+ params, GPT-4 = ~1.7T params estimated)
- Pre-trained by companies (Anthropic, Meta, Amazon, etc.)
- You DON'T train them - you USE them via API

**Amazon Bedrock** = AWS's managed service to access FMs:
```
Your App → Bedrock API → Claude/Llama/Titan → Response
```

You're essentially calling someone else's GPT-2 (but way bigger) via HTTP:
```python
# Conceptually similar to your GPT-2 inference, but via API
import boto3

bedrock = boto3.client('bedrock-runtime')
response = bedrock.invoke_model(
    modelId='anthropic.claude-3-sonnet',
    body={
        "prompt": "Explain quantum computing",
        "max_tokens": 500
    }
)
```

---

## 2. Embeddings - Converting Text to Numbers

### The Core Concept
An **embedding** is a vector (list of numbers) that represents the *meaning* of text.

```
"The cat sat on the mat" → [0.23, -0.45, 0.87, ..., 0.12]  (1536 dimensions)
"A feline rested on a rug" → [0.21, -0.43, 0.85, ..., 0.14]  (similar vector!)
"Stock prices rose today" → [-0.67, 0.92, -0.11, ..., 0.55]  (very different)
```

### How It Works (Connecting to GPT-2)
Remember the **hidden states** in your GPT-2 transformer layers? Embeddings are similar:
1. Text goes through a transformer encoder
2. Output is a fixed-size vector representing semantic meaning
3. Similar meanings = similar vectors (measured by cosine similarity)

### AWS Services for Embeddings
- **Amazon Titan Embeddings** - Amazon's embedding model
- **Cohere Embed** - Available through Bedrock
- You call these via API to convert text → vectors

```python
# Generate embedding
response = bedrock.invoke_model(
    modelId='amazon.titan-embed-text-v1',
    body={"inputText": "What is machine learning?"}
)
embedding = response['embedding']  # [0.23, -0.45, ...]
```

---

## 3. Vector Stores - The Semantic Database

### The Problem They Solve
Traditional databases search by **exact match**:
```sql
SELECT * FROM docs WHERE content LIKE '%machine learning%'
```
This misses: "ML", "artificial intelligence", "neural networks"

### Vector Stores Enable Semantic Search
Store embeddings, then find similar ones:
```
Query: "How do neural networks learn?"
        ↓ (embed)
Query Vector: [0.34, -0.21, ...]
        ↓ (find nearest neighbors)
Results: 
  1. "Backpropagation in deep learning" (similarity: 0.94)
  2. "Training AI models with gradient descent" (similarity: 0.89)
  3. "Machine learning optimization" (similarity: 0.85)
```

### AWS Vector Store Options
| Service | Description |
|---------|-------------|
| **Amazon OpenSearch** | Full-text + vector search, k-NN plugin |
| **Aurora PostgreSQL + pgvector** | Add vector search to relational DB |
| **Amazon Bedrock Knowledge Bases** | Fully managed RAG solution |
| **Amazon DynamoDB** | Metadata storage alongside vectors |

### Key Concepts for Exam
- **Indexing strategies**: How vectors are organized for fast search
- **Sharding**: Distributing vectors across nodes for scale
- **Approximate Nearest Neighbor (ANN)**: Trade accuracy for speed
- **Hybrid search**: Combine keyword + semantic search

---

## 4. RAG (Retrieval Augmented Generation) - Giving FMs Knowledge

### The Problem
FMs have a **knowledge cutoff** and don't know YOUR data:
- Claude doesn't know your company's internal docs
- It can't access real-time information
- It might "hallucinate" facts it doesn't know

### The RAG Solution
**Retrieve** relevant context, then **Augment** the prompt, then **Generate**:

```
┌──────────────────────────────────────────────────────────────────────────┐
│ 1. USER QUERY: "What's our refund policy for enterprise customers?"      │
└──────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌──────────────────────────────────────────────────────────────────────────┐
│ 2. RETRIEVE: Search vector store for relevant documents                  │
│    → Found: "Enterprise Refund Policy v2.3.pdf" (chunk 4)               │
│    → Found: "Customer Service Guidelines.docx" (chunk 12)               │
└──────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌──────────────────────────────────────────────────────────────────────────┐
│ 3. AUGMENT: Build prompt with retrieved context                          │
│                                                                          │
│    "Based on the following documents:                                    │
│     [Enterprise Refund Policy v2.3: Enterprise customers may request... │
│     [Customer Service Guidelines: For refunds exceeding $10,000...      │
│                                                                          │
│     Answer the user's question: What's our refund policy for enterprise │
│     customers?"                                                          │
└──────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌──────────────────────────────────────────────────────────────────────────┐
│ 4. GENERATE: FM produces grounded response                               │
│    "Enterprise customers have a 60-day refund window with full          │
│     reimbursement. For amounts over $10,000, approval from..."          │
└──────────────────────────────────────────────────────────────────────────┘
```

### Chunking - Breaking Documents into Pieces
Documents are too long for context windows, so we **chunk** them:

| Strategy | Description | Use Case |
|----------|-------------|----------|
| **Fixed-size** | Split every N tokens | Simple, predictable |
| **Semantic** | Split by meaning/paragraphs | Better context preservation |
| **Hierarchical** | Parent/child chunks | Complex documents |
| **Overlapping** | Chunks share some content | Avoid losing context at boundaries |

### AWS RAG Implementation
**Amazon Bedrock Knowledge Bases** handles all of this:
1. Upload docs to S3
2. Bedrock chunks them automatically
3. Generates embeddings
4. Stores in managed vector store
5. Handles retrieval at query time

---

## 5. Prompt Engineering - Controlling FM Behavior

### Why It Matters
Unlike training GPT-2 where you controlled weights, with FMs you control behavior through **prompts**.

### Key Techniques

**System Prompts** - Set the FM's role and rules:
```
You are a helpful customer service agent for Acme Corp.
Always be polite. Never discuss competitors.
If you don't know something, say so.
```

**Few-Shot Learning** - Provide examples:
```
Convert the following to SQL:

Example 1:
Question: How many users signed up last month?
SQL: SELECT COUNT(*) FROM users WHERE created_at >= DATE_SUB(NOW(), INTERVAL 1 MONTH)

Example 2:
Question: What's the average order value?
SQL: SELECT AVG(total) FROM orders

Now convert:
Question: List all products under $50
SQL:
```

**Chain-of-Thought** - Make the model reason step-by-step:
```
Solve this problem step by step:
A store has 45 apples. They sell 12 in the morning and receive 
a shipment of 30 more. How many apples do they have now?

Let's think through this:
1. Start with 45 apples
2. Sell 12: 45 - 12 = 33
3. Receive 30: 33 + 30 = 63
Answer: 63 apples
```

### AWS Prompt Management
- **Bedrock Prompt Management** - Version control for prompts
- **Bedrock Prompt Flows** - Chain prompts together visually
- **Bedrock Guardrails** - Filter harmful inputs/outputs

---

## 6. Agentic AI - The Manager/Worker Model

### YES! This is exactly like you described!

**Agentic AI** = AI systems that can:
1. **Plan** - Break down complex tasks
2. **Reason** - Decide what to do next
3. **Act** - Use tools to accomplish goals
4. **Reflect** - Evaluate results and adjust

### The Organization Analogy
```
┌─────────────────────────────────────────────────────────────────────────┐
│                         ORCHESTRATOR AGENT                              │
│                         (The Manager)                                   │
│  "User wants a market analysis report. I need to:                      │
│   1. Get stock data → assign to Data Agent                             │
│   2. Analyze trends → assign to Analysis Agent                         │
│   3. Write report → assign to Writing Agent                            │
│   4. Review and compile"                                                │
└─────────────────────────────────────────────────────────────────────────┘
            │                    │                    │
            ▼                    ▼                    ▼
    ┌───────────────┐   ┌───────────────┐   ┌───────────────┐
    │  DATA AGENT   │   │ANALYSIS AGENT │   │ WRITING AGENT │
    │  (Worker)     │   │  (Worker)     │   │  (Worker)     │
    │               │   │               │   │               │
    │ Tools:        │   │ Tools:        │   │ Tools:        │
    │ - Stock API   │   │ - Calculator  │   │ - Doc Gen     │
    │ - Database    │   │ - Charts      │   │ - Formatter   │
    └───────────────┘   └───────────────┘   └───────────────┘
```

### ReAct Pattern (Reasoning + Acting)
The most common agent pattern:
```
THOUGHT: I need to find the current stock price for AAPL
ACTION: call_stock_api(symbol="AAPL")
OBSERVATION: {"price": 178.50, "change": "+2.3%"}
THOUGHT: Now I have the price. User also asked for comparison to last month.
ACTION: call_stock_api(symbol="AAPL", period="1M")
OBSERVATION: {"price_1m_ago": 165.20}
THOUGHT: I can now calculate the monthly change and respond.
FINAL ANSWER: AAPL is currently at $178.50, up 8.05% from last month's $165.20.
```

### AWS Agentic Services
| Service | Purpose |
|---------|---------|
| **Amazon Bedrock Agents** | Managed agents with tool use |
| **AWS Strands Agents** | Open-source agent framework |
| **AWS Agent Squad** | Multi-agent orchestration |
| **AWS Step Functions** | Workflow orchestration (state machines) |
| **Model Context Protocol (MCP)** | Standard for agent-tool communication |

### Tools = Functions Agents Can Call
```python
# Define a tool for the agent
tools = [
    {
        "name": "get_weather",
        "description": "Get current weather for a location",
        "parameters": {
            "location": {"type": "string", "description": "City name"}
        }
    },
    {
        "name": "send_email",
        "description": "Send an email to a recipient",
        "parameters": {
            "to": {"type": "string"},
            "subject": {"type": "string"},
            "body": {"type": "string"}
        }
    }
]

# Agent decides which tool to use based on user request
# "What's the weather in Seattle?" → calls get_weather("Seattle")
# "Email John about the meeting" → calls send_email(...)
```

---

## 7. How It All Connects - A Complete Example

**Scenario**: User asks a customer service bot: "Can I return my order #12345?"

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ STEP 1: User Input                                                          │
│ "Can I return my order #12345?"                                             │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ STEP 2: Agent Reasoning (Bedrock Agent)                                     │
│ THOUGHT: User wants to return order. I need to:                             │
│   1. Look up order details                                                  │
│   2. Check return policy                                                    │
│   3. Determine if eligible                                                  │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ STEP 3: Tool Call - Order Lookup                                            │
│ ACTION: get_order_details(order_id="12345")                                 │
│ RESULT: {product: "Laptop", date: "2024-12-01", status: "delivered"}        │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ STEP 4: RAG - Policy Retrieval (Bedrock Knowledge Base)                     │
│ Query: "laptop return policy"                                               │
│ Retrieved: "Electronics may be returned within 30 days of delivery..."     │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ STEP 5: Agent Reasoning                                                     │
│ THOUGHT: Order delivered Dec 1, today is Dec 22. That's 21 days.           │
│          Policy allows 30 days. Customer IS eligible for return.           │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ STEP 6: Generate Response (Foundation Model via Bedrock)                    │
│ "Yes! Your laptop order #12345 is eligible for return. You have 9 days     │
│  remaining in your 30-day return window. Would you like me to initiate     │
│  the return process?"                                                       │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 8. Comparison: Your GPT-2 Work vs AWS GenAI

| Aspect | Your GPT-2 Project | AWS GenAI (AIP-C01) |
|--------|-------------------|---------------------|
| **Model** | Train/fine-tune yourself | Use pre-trained FMs via API |
| **Scale** | ~1.5B params, single GPU | 100B+ params, managed infrastructure |
| **Customization** | Modify weights | Prompt engineering, fine-tuning via API |
| **Knowledge** | In training data only | RAG adds external knowledge |
| **Actions** | Generate text only | Agents can use tools, APIs, databases |
| **Deployment** | Your own infrastructure | Bedrock handles scaling, availability |
| **Cost model** | Compute time | Per-token pricing |

---

## Key AWS Services to Know for Exam

### Core GenAI Services
- **Amazon Bedrock** - Managed FM access (Claude, Llama, Titan, etc.)
- **Amazon SageMaker AI** - Custom model training/deployment
- **Amazon Q** - AWS's AI assistant (Q Developer, Q Business)

### Supporting Services
- **Amazon OpenSearch** - Vector search
- **Amazon Aurora + pgvector** - Relational DB with vectors
- **AWS Lambda** - Serverless compute for processing
- **AWS Step Functions** - Workflow orchestration
- **Amazon S3** - Document storage
- **Amazon CloudWatch** - Monitoring and logging
- **AWS IAM** - Security and access control
- **Amazon Comprehend** - NLP (entity extraction, PII detection)
- **Amazon Transcribe** - Speech to text

---

## Next Steps for Study

1. **Hands-on**: Build a simple RAG app with Bedrock Knowledge Bases
2. **Practice Tests**: Use Tutorials Dojo free sampler
3. **Deep Dive**: Focus on Domain 1 (31%) and Domain 2 (26%)
4. **Understand Guardrails**: Safety is 20% of the exam
