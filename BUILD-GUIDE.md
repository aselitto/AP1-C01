# Build Your Own AIP-C01 Training Chatbot

A step-by-step guide to building a RAG-powered study assistant using Amazon Bedrock. By building this, you'll gain hands-on experience with the exact services tested on the AP1-C01 exam.

## 🎯 What You'll Build

A chatbot that:
1. **Asks you exam questions** from our 900-question database
2. **Evaluates your answers** and tells you if you're correct
3. **Explains why** using RAG retrieval from the question explanations
4. **Tracks interventions** via CloudWatch (guardrails monitoring)

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     AP1-C01 Training Chatbot                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   S3 Bucket                                                      │
│   └── questions/                                                 │
│       └── parsed_questions.json    ◄── Upload this file         │
│                     │                                            │
│                     ▼                                            │
│   ┌─────────────────────────────────┐                           │
│   │   Bedrock Knowledge Base        │                           │
│   │   - Titan Embeddings V2         │                           │
│   │   - OpenSearch Serverless       │                           │
│   └─────────────────────────────────┘                           │
│                     │                                            │
│                     ▼                                            │
│   ┌─────────────────────────────────┐                           │
│   │   Bedrock Guardrails            │                           │
│   │   - Content filters (Medium)    │                           │
│   │   - Denied topics               │                           │
│   └─────────────────────────────────┘                           │
│                     │                                            │
│                     ▼                                            │
│   ┌─────────────────────────────────┐                           │
│   │   Claude 3.5 Sonnet / Haiku     │                           │
│   │   (via Converse API)            │                           │
│   └─────────────────────────────────┘                           │
│                     │                                            │
│                     ▼                                            │
│   ┌─────────────────────────────────┐                           │
│   │   CloudWatch Metrics            │                           │
│   │   - InvocationsIntervened       │                           │
│   │   - Latency tracking            │                           │
│   └─────────────────────────────────┘                           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## 📋 Prerequisites

- AWS Account with Bedrock access enabled
- Model access enabled for:
  - Claude 3.5 Sonnet (or Claude 3 Haiku for lower cost)
  - Amazon Titan Text Embeddings V2
- Basic familiarity with AWS Console

## 🔧 Step-by-Step Build Instructions

---

### Step 1: Create S3 Bucket for Questions

**Why:** Knowledge Bases need a data source. S3 is the most common choice.

1. Go to **S3 Console** → **Create bucket**
2. **Bucket name:** `aip-c01-training-data-{your-account-id}`
3. **Region:** Same region as your Bedrock access (e.g., us-east-1)
4. Keep defaults → **Create bucket**

5. **Upload the questions:**
   - Click into your bucket
   - Create folder: `questions/`
   - Upload `parsed_questions.json` to this folder

> 💡 **Exam Tip:** S3 is the primary data source for Knowledge Bases. You can also use Confluence, SharePoint, or Web Crawler.

---

### Step 2: Create the Knowledge Base

**Why:** This enables RAG - retrieving relevant question explanations based on user queries.

1. Go to **Amazon Bedrock Console** → **Knowledge bases** → **Create knowledge base**

2. **Knowledge base details:**
   - **Name:** `aip-c01-training-kb`
   - **Description:** `Practice questions and explanations for AIP-C01 exam`
   - **IAM permissions:** Create and use a new service role

3. **Choose data source:**
   - Select **Amazon S3**
   - Click **Next**

4. **Configure data source:**
   - **Data source name:** `exam-questions`
   - **S3 URI:** Browse to your bucket → `s3://aip-c01-training-data-xxx/questions/`

5. **Chunking strategy:**
   - Select **Default chunking**
   - (For production, you might use custom chunking via Lambda)

6. **Embeddings model:**
   - Select **Titan Text Embeddings V2**
   - **Vector dimensions:** 1024 (default)

7. **Vector database:**
   - Select **Quick create a new vector store**
   - This creates an OpenSearch Serverless collection automatically

8. Click **Create knowledge base**

9. **Wait for sync** - this may take 5-10 minutes

10. **Sync the data source:**
    - Once created, click on your knowledge base
    - Go to **Data sources** tab
    - Select your data source → **Sync**

> 💡 **Exam Tip:** When embedding models change, you MUST re-sync/re-index. Old vectors become incompatible!

---

### Step 3: Create Guardrails

**Why:** Demonstrates content filtering and safety controls (Domain 3 of the exam).

1. Go to **Amazon Bedrock Console** → **Guardrails** → **Create guardrail**

2. **Step 1 - Guardrail details:**
   - **Name:** `aip-c01-training-guardrail`
   - **Blocked message:** `This question is outside the scope of AIP-C01 exam preparation.`

3. **Step 2 - Content filters:**
   - Enable **Harmful categories filters**
   - Set all categories (Hate, Insults, Sexual, Violence, Misconduct) to **Medium**
   - Action: **Block**

4. **Step 3 - Denied topics:**
   - Click **Add denied topic**
   - **Name:** `off-topic-questions`
   - **Definition:** `Questions not related to AWS, cloud computing, AI/ML, or the AIP-C01 certification exam`
   - **Input action:** Block
   - **Output action:** Block
   - Click **Confirm**

5. **Step 4 - Word filters:**
   - Skip (click Next)

6. **Step 5 - Sensitive information filters:**
   - Click **Add new PII**
   - Add **EMAIL** with action **Mask**
   - Add **PHONE** with action **Mask**
   - Click **Confirm** for each

7. **Step 6 - Contextual grounding:**
   - Skip for now (more useful with complex RAG)

8. **Step 7 - Automated Reasoning:**
   - Skip

9. **Review and create**

> 💡 **Exam Tip:** 
> - **BLOCK** = Stops entire request
> - **MASK/ANONYMIZE** = Replaces PII with placeholder, conversation continues
> - Use ANONYMIZE + tokenization when you need to recover original PII later

---

### Step 4: Test in the Playground

**Why:** Verify everything works before building an application.

1. Go to **Amazon Bedrock Console** → **Playgrounds** → **Chat**

2. **Select model:**
   - Choose **Claude 3.5 Sonnet** (or Haiku for lower cost)

3. **Configure Knowledge Base:**
   - Click **Select knowledge base**
   - Choose `aip-c01-training-kb`

4. **Configure Guardrails:**
   - Click **Select guardrail**
   - Choose `aip-c01-training-guardrail`
   - Select the latest version

5. **Test with prompts:**

   ```
   Ask me a practice question about Amazon Bedrock Guardrails
   ```

   ```
   What's the difference between BLOCK and ANONYMIZE modes for PII?
   ```

   ```
   Give me a question about SageMaker endpoint types
   ```

6. **Verify RAG is working:**
   - Look for citations/references in the response
   - The model should cite specific questions from your database

> 💡 **Exam Tip:** The Converse API maintains context via the `messages` array - you must pass the full conversation history with each call. There's no automatic session management.

---

### Step 5: Monitor with CloudWatch

**Why:** Domain 4 covers monitoring and operational efficiency.

1. Go to **CloudWatch Console** → **Metrics** → **All metrics**

2. Search for **Bedrock**

3. Key metrics to monitor:
   - `InvocationsIntervened` - How often guardrails blocked content
   - `Invocations` - Total API calls
   - `InvocationLatency` - Response time

4. **Create a dashboard:**
   - Click **Dashboards** → **Create dashboard**
   - Name: `AIP-C01-Training-Metrics`
   - Add widgets for the metrics above

5. **Filter by guardrail:**
   - Use dimension `GuardrailId` to see your specific guardrail
   - Use `GuardrailPolicyType` to see which policy triggered (ContentFilter, DeniedTopic, etc.)

> 💡 **Exam Tip:** CloudWatch metrics for guardrails include `InvocationsIntervened` with dimensions for `GuardrailPolicyType` - this tells you WHICH policy triggered the intervention.

---

### Step 6: (Optional) Build an API

For a production-ready chatbot, you'd add:

1. **API Gateway** - REST API endpoint
2. **Lambda** - Backend logic using Bedrock SDK
3. **DynamoDB** - Store conversation history and user progress

**Lambda code example (Python):**

```python
import boto3
import json

bedrock = boto3.client('bedrock-runtime')
bedrock_agent = boto3.client('bedrock-agent-runtime')

def lambda_handler(event, context):
    user_message = event['body']['message']
    
    # Retrieve from Knowledge Base
    kb_response = bedrock_agent.retrieve(
        knowledgeBaseId='YOUR_KB_ID',
        retrievalQuery={'text': user_message},
        retrievalConfiguration={
            'vectorSearchConfiguration': {
                'numberOfResults': 5
            }
        }
    )
    
    # Build context from retrieved documents
    context_text = "\n".join([
        r['content']['text'] for r in kb_response['retrievalResults']
    ])
    
    # Call model with Converse API
    response = bedrock.converse(
        modelId='anthropic.claude-3-sonnet-20240229-v1:0',
        messages=[
            {
                'role': 'user',
                'content': [{'text': f"Context: {context_text}\n\nQuestion: {user_message}"}]
            }
        ],
        guardrailConfig={
            'guardrailIdentifier': 'YOUR_GUARDRAIL_ID',
            'guardrailVersion': 'DRAFT'
        }
    )
    
    return {
        'statusCode': 200,
        'body': json.dumps({
            'response': response['output']['message']['content'][0]['text']
        })
    }
```

---

## ✅ What You've Learned

By building this chatbot, you've covered:

| Exam Domain | What You Did |
|-------------|--------------|
| **Domain 1** (31%) | Created Knowledge Base, configured embeddings, set up S3 data source |
| **Domain 2** (26%) | Used Converse API, integrated Knowledge Base with model |
| **Domain 3** (20%) | Configured Guardrails with content filters, PII masking, denied topics |
| **Domain 4** (14%) | Set up CloudWatch monitoring for guardrail interventions |
| **Domain 5** (12%) | Tested in Playground, validated RAG retrieval quality |

---

## 🧹 Cleanup

To avoid ongoing charges:

1. **Delete Knowledge Base:**
   - Bedrock Console → Knowledge bases → Select → Delete
   - This also deletes the OpenSearch Serverless collection

2. **Delete S3 Bucket:**
   - Empty the bucket first
   - Then delete the bucket

3. **Delete Guardrail:**
   - Bedrock Console → Guardrails → Select → Delete

4. **Delete CloudWatch Dashboard:**
   - CloudWatch → Dashboards → Select → Delete

---

## 🎓 Next Steps

1. **Add more questions** - Expand the question database
2. **Track progress** - Add DynamoDB to store user scores by domain
3. **Add difficulty levels** - Tag questions and filter by difficulty
4. **Deploy publicly** - Add Cognito authentication for team access

---

## 📚 Related Exam Topics

This project demonstrates these frequently-tested concepts:

- **Converse API** vs InvokeModel (Converse is model-agnostic!)
- **Knowledge Base chunking strategies** (default, fixed, semantic, custom Lambda)
- **Guardrail modes** (BLOCK vs ANONYMIZE/Mask)
- **CloudWatch metrics** for Bedrock (InvocationsIntervened, latency)
- **Vector store options** (OpenSearch Serverless auto-created here)
- **Embedding model selection** (Titan V2 with configurable dimensions)

---

*Good luck building and studying! 🚀*
