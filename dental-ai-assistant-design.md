# Dental AI Response Evaluation System

## Complete Design & Implementation Document

---

# PART 1: SYSTEM UNDERSTANDING (Simple Language)

---

## What Does This System Do?

```
┌─────────────────────────────────────────────────────────────────────────┐
│                                                                         │
│   DENTIST creates questions     ──▶   PATIENT receives questions        │
│                                                                         │
│   PATIENT responds via AUDIO    ──▶   AI EVALUATES the response         │
│                                                                         │
│   AI compares with CORRECT      ──▶   RESULT shared with                │
│   ANSWERS from Knowledge Base         PATIENT & DENTIST                 │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Simple Example

| Step | What Happens |
|------|-------------|
| 1 | Dentist creates a question: "What should you avoid after tooth extraction?" |
| 2 | Dentist uploads the correct answer to the Knowledge Base |
| 3 | Patient receives the question on their app |
| 4 | Patient records an audio answer: "Avoid hot food, don't use straw, no smoking" |
| 5 | System converts audio to text (Speech-to-Text) |
| 6 | AI fetches the correct answer from Knowledge Base |
| 7 | AI compares patient's answer vs correct answer |
| 8 | AI generates score + feedback: "3/6 — Good! But also avoid hard food, spitting, alcohol" |
| 9 | Patient and Dentist both see the result |

---

## How Does the AI Evaluation Work?

This is the core of the system. Here's how AI evaluates a patient's audio response:

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                         AI EVALUATION PROCESS                                    │
│                                                                                  │
│                                                                                  │
│   ┌───────────────────┐                                                         │
│   │  PATIENT'S AUDIO  │                                                         │
│   │  "Avoid hot food, │                                                         │
│   │   no straw, no    │                                                         │
│   │   smoking"        │                                                         │
│   └────────┬──────────┘                                                         │
│            │                                                                     │
│            ▼                                                                     │
│   ┌───────────────────┐                                                         │
│   │  STEP 1:          │                                                         │
│   │  SPEECH-TO-TEXT   │  Amazon Transcribe converts audio to text               │
│   │                   │                                                         │
│   │  Output: "Avoid   │                                                         │
│   │  hot food, don't  │                                                         │
│   │  use straw, no    │                                                         │
│   │  smoking"         │                                                         │
│   └────────┬──────────┘                                                         │
│            │                                                                     │
│            ▼                                                                     │
│   ┌───────────────────┐         ┌──────────────────────────────────┐           │
│   │  STEP 2:          │         │  KNOWLEDGE BASE (Your Data)      │           │
│   │  RETRIEVE         │◀────────│                                   │           │
│   │  CORRECT ANSWER   │         │  Contains correct answers that    │           │
│   │                   │         │  dentist uploaded:                 │           │
│   │  AI searches KB   │         │                                   │           │
│   │  for the matching │         │  • Avoid hot foods ✓              │           │
│   │  reference answer │         │  • No hard/crunchy foods          │           │
│   └────────┬──────────┘         │  • Don't use straws ✓            │           │
│            │                     │  • No smoking ✓                   │           │
│            │                     │  • No forceful spitting           │           │
│            ▼                     │  • No alcohol                     │           │
│   ┌───────────────────┐         └──────────────────────────────────┘           │
│   │  STEP 3:          │                                                         │
│   │  AI COMPARISON    │                                                         │
│   │                   │                                                         │
│   │  Claude AI takes: │                                                         │
│   │  • Patient text   │                                                         │
│   │  • Correct answer │                                                         │
│   │                   │                                                         │
│   │  And compares     │                                                         │
│   │  them point by    │                                                         │
│   │  point            │                                                         │
│   └────────┬──────────┘                                                         │
│            │                                                                     │
│            ▼                                                                     │
│   ┌───────────────────────────────────────────────────────────────┐             │
│   │  STEP 4: RESULT                                                │             │
│   │                                                                 │             │
│   │  Score: 3/6 (50%)                                              │             │
│   │                                                                 │             │
│   │  ✅ Correct: Hot foods, Straws, Smoking                        │             │
│   │  ❌ Missed:  Hard foods, Forceful spitting, Alcohol            │             │
│   │                                                                 │             │
│   │  Feedback: "Good job identifying hot foods, straws, and         │             │
│   │  smoking! Also remember to avoid hard/crunchy foods,            │             │
│   │  forceful spitting, and alcohol for 24 hours."                  │             │
│   └───────────────────────────────────────────────────────────────┘             │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## What is the Knowledge Base (RAG)?

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                                                                                  │
│   DENTIST uploads:                    SYSTEM stores it as:                        │
│                                                                                  │
│   ┌─────────────────────┐            ┌─────────────────────────────────┐        │
│   │ • Correct answers   │            │  Amazon Bedrock Knowledge Base   │        │
│   │ • Scoring rubrics   │  ────▶     │                                  │        │
│   │ • Dental guides     │            │  Converts text into "embeddings" │        │
│   │ • PDF documents     │            │  (numerical search index)        │        │
│   └─────────────────────┘            │                                  │        │
│                                       │  When AI needs the correct       │        │
│                                       │  answer, it searches here        │        │
│                                       │  and finds the best match        │        │
│                                       └─────────────────────────────────┘        │
│                                                                                  │
│   WHY? So AI doesn't guess — it uses YOUR specific correct answers              │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## Complete End-to-End Flow (All Steps)

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                     COMPLETE SYSTEM FLOW                                          │
└─────────────────────────────────────────────────────────────────────────────────┘

 ══════════════════════════════════════════════════════════════════════════════════
  PHASE A: SETUP (One-time by Dentist)
 ══════════════════════════════════════════════════════════════════════════════════

  DENTIST
     │
     ├──① Upload correct answers & dental docs ──▶ S3 Bucket ──▶ Bedrock KB
     │                                                           (indexed for search)
     │
     ├──② Create questions ──▶ API ──▶ DynamoDB (stored)
     │
     └──③ Assign questions to patients ──▶ API ──▶ DynamoDB
                                                       │
                                                       ▼
                                              Patient gets notification
                                              (SMS/Email/Push)


 ══════════════════════════════════════════════════════════════════════════════════
  PHASE B: PATIENT RESPONSE
 ══════════════════════════════════════════════════════════════════════════════════

  PATIENT
     │
     ├──④ Opens app, sees assigned questions
     │
     ├──⑤ Records audio/video answer
     │
     └──⑥ Uploads recording ──▶ S3 (audio stored securely)
                                    │
                                    ▼
                           Upload triggers pipeline
                           (automatic — no manual step)


 ══════════════════════════════════════════════════════════════════════════════════
  PHASE C: AI PROCESSING (Fully Automatic)
 ══════════════════════════════════════════════════════════════════════════════════

  S3 Upload Event
     │
     ▼
  ┌──────────────────────────────────────────────────────────────────────┐
  │  AWS STEP FUNCTIONS (Automated Pipeline)                              │
  │                                                                       │
  │  ⑦ VALIDATE ──▶ ⑧ TRANSCRIBE ──▶ ⑨ EVALUATE ──▶ ⑩ STORE ──▶ ⑪ NOTIFY│
  │                                                                       │
  │  ⑦ Check file    ⑧ Audio → Text   ⑨ Compare with  ⑩ Save     ⑪ Send │
  │    is valid        using Amazon      Knowledge Base   results    alert │
  │    (type, size)    Transcribe        + Score with     to DB      to   │
  │                                      Claude AI                  users │
  └──────────────────────────────────────────────────────────────────────┘


 ══════════════════════════════════════════════════════════════════════════════════
  PHASE D: RESULTS
 ══════════════════════════════════════════════════════════════════════════════════

  PATIENT                                    DENTIST
     │                                          │
     ├──⑫ Gets notification                    ├──⑫ Gets email summary
     │     "Your results are ready"             │
     │                                          │
     ├──⑬ Opens app → Views:                   ├──⑬ Opens dashboard → Views:
     │     • Score (3/6)                        │     • All patient scores
     │     • What they got right                │     • Trends over time
     │     • What they missed                   │     • Patients needing attention
     │     • Helpful feedback                   │     • Export reports
     │                                          │
     └──────────────────────────────────────────└─────────────────────────────
```

---

## Detailed Step ⑨: The Evaluation (Most Important Step)

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│  STEP ⑨ — AI EVALUATION (Inside evaluate-response Lambda)                       │
│                                                                                  │
│                                                                                  │
│  INPUT: Patient's transcribed text                                               │
│  "I should avoid hot food and not use a straw. Also no smoking."                │
│                                                                                  │
│         ┌─────────────────────────────────────────────────────────┐             │
│         │                                                          │             │
│         │  SUB-STEP A: Query Knowledge Base                        │             │
│         │                                                          │             │
│         │  → Send the question text to Bedrock Knowledge Base      │             │
│         │  → KB searches through all uploaded dental documents      │             │
│         │  → Returns the matching reference answer:                 │             │
│         │                                                          │             │
│         │    "Patients should avoid: hot foods, hard/crunchy       │             │
│         │     foods, straws, smoking, forceful spitting, alcohol"   │             │
│         │                                                          │             │
│         └──────────────────────────┬───────────────────────────────┘             │
│                                    │                                              │
│                                    ▼                                              │
│         ┌─────────────────────────────────────────────────────────┐             │
│         │                                                          │             │
│         │  SUB-STEP B: Send to Claude AI for Comparison            │             │
│         │                                                          │             │
│         │  Claude receives:                                         │             │
│         │    1. The question                                        │             │
│         │    2. The correct answer (from KB)                        │             │
│         │    3. The patient's answer (from Transcribe)              │             │
│         │                                                          │             │
│         │  Claude compares point-by-point and returns:              │             │
│         │    • Score: 3 out of 6                                    │             │
│         │    • Correct: [hot foods, straws, smoking]                │             │
│         │    • Missed: [hard foods, spitting, alcohol]              │             │
│         │    • Feedback: "Good job! Also remember..."               │             │
│         │                                                          │             │
│         └──────────────────────────┬───────────────────────────────┘             │
│                                    │                                              │
│                                    ▼                                              │
│         ┌─────────────────────────────────────────────────────────┐             │
│         │                                                          │             │
│         │  SUB-STEP C: Format and Return Result                    │             │
│         │                                                          │             │
│         │  {                                                        │             │
│         │    "score": 3,                                            │             │
│         │    "total_points": 6,                                     │             │
│         │    "percentage": 50,                                      │             │
│         │    "correct_points": [                                    │             │
│         │      "Avoid hot foods",                                   │             │
│         │      "Do not use straws",                                 │             │
│         │      "No smoking"                                         │             │
│         │    ],                                                     │             │
│         │    "missed_points": [                                     │             │
│         │      "No hard/crunchy foods",                             │             │
│         │      "Avoid forceful spitting",                           │             │
│         │      "No alcohol"                                         │             │
│         │    ],                                                     │             │
│         │    "feedback": "Good job identifying hot foods,           │             │
│         │     straws, and smoking! Also remember to avoid           │             │
│         │     hard foods, forceful spitting, and alcohol."          │             │
│         │  }                                                        │             │
│         │                                                          │             │
│         └─────────────────────────────────────────────────────────┘             │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## All AWS Services — Visual Map

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                                                                                  │
│                        AWS SERVICES MAP                                           │
│                                                                                  │
│  ┌─────────────────────────────────────────────────────────────────────────┐    │
│  │  USER-FACING                                                             │    │
│  │                                                                          │    │
│  │  CloudFront ─── Hosts web apps (dentist + patient)                      │    │
│  │  Cognito ────── Login/signup (email, password, OTP)                     │    │
│  │  API Gateway ── All API endpoints (REST)                                │    │
│  └─────────────────────────────────────────────────────────────────────────┘    │
│                                                                                  │
│  ┌─────────────────────────────────────────────────────────────────────────┐    │
│  │  BACKEND LOGIC                                                           │    │
│  │                                                                          │    │
│  │  Lambda ─────── All business logic (13 functions)                       │    │
│  │  Step Functions  Pipeline orchestration (validate → transcribe →        │    │
│  │                  evaluate → store → notify)                              │    │
│  │  EventBridge ── Triggers pipeline when file uploaded                    │    │
│  └─────────────────────────────────────────────────────────────────────────┘    │
│                                                                                  │
│  ┌─────────────────────────────────────────────────────────────────────────┐    │
│  │  DATA STORAGE                                                            │    │
│  │                                                                          │    │
│  │  S3 ──────────── Audio/video files + Knowledge Base documents           │    │
│  │  DynamoDB ────── Questions, responses, results, users                   │    │
│  └─────────────────────────────────────────────────────────────────────────┘    │
│                                                                                  │
│  ┌─────────────────────────────────────────────────────────────────────────┐    │
│  │  AI / MACHINE LEARNING                                                   │    │
│  │                                                                          │    │
│  │  Transcribe ──── Converts audio/video → text                            │    │
│  │  Bedrock KB ──── Stores & retrieves correct answers (RAG)               │    │
│  │  Bedrock Claude ─ Evaluates patient answer vs correct answer            │    │
│  └─────────────────────────────────────────────────────────────────────────┘    │
│                                                                                  │
│  ┌─────────────────────────────────────────────────────────────────────────┐    │
│  │  NOTIFICATIONS                                                           │    │
│  │                                                                          │    │
│  │  SNS ──────────── Push notifications + SMS                              │    │
│  │  SES ──────────── Email notifications                                   │    │
│  └─────────────────────────────────────────────────────────────────────────┘    │
│                                                                                  │
│  ┌─────────────────────────────────────────────────────────────────────────┐    │
│  │  SECURITY & MONITORING                                                   │    │
│  │                                                                          │    │
│  │  WAF ──────────── Protects API from attacks                             │    │
│  │  KMS ──────────── Encrypts all data                                     │    │
│  │  IAM ──────────── Controls who can access what                          │    │
│  │  CloudWatch ───── Logs, metrics, alarms                                 │    │
│  └─────────────────────────────────────────────────────────────────────────┘    │
│                                                                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---


---

# PART 2: TECHNICAL ARCHITECTURE

---

## Complete System Architecture Diagram

```
┌────────────────────────┐                         ┌────────────────────────┐
│   DENTIST ADMIN APP    │                         │   PATIENT APP          │
│   (React/Next.js)      │                         │   (React Native/Web)   │
│                        │                         │                        │
│  • Create questions    │                         │  • View questions      │
│  • Upload KB docs      │                         │  • Record audio/video  │
│  • View results        │                         │  • View results        │
│  • Manage patients     │                         │  • Get feedback        │
└───────────┬────────────┘                         └───────────┬────────────┘
            │                                                  │
            ▼                                                  ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         AWS CloudFront (CDN)                                 │
│                    Static hosting + HTTPS + Caching                          │
└─────────────────────────────────────────────────────────────────────────────┘
            │                                                  │
            ▼                                                  ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         Amazon Cognito                                       │
│               User Pools (dentist/patient authentication)                    │
│               JWT tokens, MFA, password policies                             │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         Amazon API Gateway (REST)                            │
│                                                                             │
│  Dentist Endpoints:                    Patient Endpoints:                    │
│  POST /questions                       GET  /my/questions/{assignmentId}    │
│  POST /knowledge-base/upload           POST /my/responses/upload-url        │
│  GET  /results/{patientId}             POST /my/responses                   │
│  GET  /dashboard/stats                 GET  /my/results                     │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         AWS Lambda Functions                                 │
│                                                                             │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐     │
│  │  Question    │ │  Upload URL  │ │  KB Manager  │ │  Results     │     │
│  │  Manager    │ │  Generator   │ │              │ │  Fetcher     │     │
│  └──────────────┘ └──────────────┘ └──────────────┘ └──────────────┘     │
└─────────────────────────────────────────────────────────────────────────────┘
         │                    │                    │                │
         ▼                    ▼                    ▼                ▼
┌──────────────┐    ┌──────────────┐    ┌──────────────────────────────────┐
│  DynamoDB    │    │  S3 Buckets  │    │  AWS Step Functions              │
│              │    │              │    │  (Evaluation Pipeline)            │
│ • Questions  │    │ • Audio files│    │                                   │
│ • Responses  │    │ • KB docs    │    │  Validate → Transcribe →         │
│ • Results    │    │ • Transcripts│    │  Evaluate → Store → Notify       │
│ • Users      │    │              │    │                                   │
└──────────────┘    └──────────────┘    └───────────┬──────────────────────┘
                                                    │
                                    ┌───────────────┼───────────────┐
                                    ▼               ▼               ▼
                          ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
                          │  Amazon      │ │  Bedrock     │ │  Bedrock     │
                          │  Transcribe  │ │  Knowledge   │ │  Claude AI   │
                          │              │ │  Base (RAG)  │ │  (Evaluator) │
                          │  Audio→Text  │ │  Your answers│ │  Compares &  │
                          │              │ │  stored here │ │  scores      │
                          └──────────────┘ └──────────────┘ └──────────────┘
```

---

## API Design

### Dentist APIs

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | /questions | Create a new question |
| GET | /questions | List all questions |
| PUT | /questions/{id} | Update a question |
| DELETE | /questions/{id} | Delete a question |
| POST | /assignments | Assign questions to patient(s) |
| GET | /results | Get all patient results (with filters) |
| GET | /results/{patientId} | Get specific patient results |
| POST | /knowledge-base/upload | Upload KB documents |
| GET | /dashboard/stats | Aggregated statistics |

### Patient APIs

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | /my/assignments | Get assigned question sets |
| GET | /my/questions/{assignmentId} | Get questions for an assignment |
| POST | /my/responses/upload-url | Get presigned URL for audio/video upload |
| POST | /my/responses | Confirm upload + trigger processing |
| GET | /my/results | Get own results/feedback |

---

## Database Schema (DynamoDB)

### Questions Table

```
PK: QUESTION#<questionId>
SK: META

Attributes:
- questionId (String, UUID)
- dentistId (String)
- questionText (String)
- category (String: "post-extraction", "hygiene", "orthodontics", etc.)
- difficulty (String: "easy" | "medium" | "hard")
- expectedKeyPoints (List of Strings)
- totalPoints (Number)
- passingScore (Number)
- createdAt (ISO 8601)

GSI: dentistId-index (PK: dentistId, SK: createdAt)
```

### Assignments Table

```
PK: ASSIGNMENT#<assignmentId>
SK: PATIENT#<patientId>

Attributes:
- assignmentId (String, UUID)
- dentistId (String)
- patientId (String)
- questionIds (List of Strings)
- status (String: "sent" | "in-progress" | "completed")
- dueDate (ISO 8601)
- sentAt (ISO 8601)

GSI: patientId-index (PK: patientId, SK: sentAt)
```

### Responses Table

```
PK: RESPONSE#<responseId>
SK: QUESTION#<questionId>

Attributes:
- responseId (String, UUID)
- patientId (String)
- assignmentId (String)
- questionId (String)
- s3Key (String)
- fileType (String: "audio/webm", "video/mp4", etc.)
- duration (Number — seconds)
- transcript (String — filled after Transcribe)
- status (String: "uploaded" | "transcribing" | "evaluating" | "completed" | "failed")
- uploadedAt (ISO 8601)

GSI: patientId-index (PK: patientId, SK: uploadedAt)
GSI: status-index (PK: status, SK: uploadedAt)
```

### Results Table

```
PK: RESULT#<resultId>
SK: PATIENT#<patientId>

Attributes:
- resultId (String, UUID)
- responseId (String)
- patientId (String)
- questionId (String)
- score (Number)
- totalPoints (Number)
- percentage (Number)
- correctPoints (List of Strings)
- missedPoints (List of Strings)
- feedback (String — AI-generated)
- evaluatedAt (ISO 8601)

GSI: patientId-index (PK: patientId, SK: evaluatedAt)
```

---

## S3 Bucket Structure

```
dental-ai-responses-{env}/
├── responses/{patientId}/{responseId}.webm
├── transcripts/{responseId}.json
└── temp/

dental-ai-knowledge-{env}/
├── reference-answers/{questionId}.json
├── scoring-rubrics/{category}-rubric.json
└── dental-guides/{topic}.pdf
```

---

## Processing Pipeline (Step Functions State Machine)

```json
{
  "Comment": "Dental AI Response Evaluation Pipeline",
  "StartAt": "ValidateInput",
  "States": {
    "ValidateInput": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:...:validate-input",
      "Next": "TranscribeAudio",
      "Catch": [{ "ErrorEquals": ["ValidationError"], "Next": "MarkFailed" }]
    },
    "TranscribeAudio": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:...:start-transcription",
      "Next": "WaitForTranscription"
    },
    "WaitForTranscription": {
      "Type": "Wait",
      "Seconds": 30,
      "Next": "CheckTranscriptionStatus"
    },
    "CheckTranscriptionStatus": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:...:check-transcription",
      "Next": "IsTranscriptionComplete"
    },
    "IsTranscriptionComplete": {
      "Type": "Choice",
      "Choices": [
        { "Variable": "$.transcriptionStatus", "StringEquals": "COMPLETED", "Next": "EvaluateResponse" },
        { "Variable": "$.transcriptionStatus", "StringEquals": "FAILED", "Next": "MarkFailed" }
      ],
      "Default": "WaitForTranscription"
    },
    "EvaluateResponse": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:...:evaluate-response",
      "Next": "StoreResults"
    },
    "StoreResults": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:...:store-results",
      "Next": "SendNotifications"
    },
    "SendNotifications": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:...:send-notifications",
      "End": true
    },
    "MarkFailed": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:...:mark-failed",
      "End": true
    }
  }
}
```

---

## AI Evaluation Implementation Code

### Step 1: Retrieve Reference Answer from Knowledge Base

```python
import boto3

bedrock_agent = boto3.client('bedrock-agent-runtime')

def retrieve_reference(question_text):
    """
    Searches the Knowledge Base for the correct/reference answer
    that matches the question being evaluated.
    """
    response = bedrock_agent.retrieve(
        knowledgeBaseId='YOUR_KB_ID',
        retrievalQuery={
            'text': question_text
        },
        retrievalConfiguration={
            'vectorSearchConfiguration': {
                'numberOfResults': 3
            }
        }
    )
    
    # Combine top results as reference context
    reference_texts = []
    for result in response['retrievalResults']:
        reference_texts.append(result['content']['text'])
    
    return "\n".join(reference_texts)
```

### Step 2: Evaluate Patient Response Against Reference

```python
import boto3
import json

bedrock = boto3.client('bedrock-runtime')

def evaluate_response(transcript, reference_answer, question_text):
    """
    Sends the patient's transcribed response + the correct reference answer
    to Claude AI, which compares them and returns a structured evaluation.
    """
    prompt = f"""You are a dental health knowledge evaluator.

QUESTION ASKED:
{question_text}

REFERENCE ANSWER (correct/expected response from Knowledge Base):
{reference_answer}

PATIENT'S RESPONSE (transcribed from their audio recording):
{transcript}

EVALUATION TASK:
1. Compare the patient's response against the reference answer
2. Identify which key points were correctly mentioned
3. Identify which key points were missed
4. Calculate a score (number of correct points / total expected points)
5. Provide brief, encouraging, patient-friendly feedback

Respond ONLY in this JSON format:
{{
  "score": <number>,
  "total_points": <number>,
  "percentage": <number>,
  "correct_points": ["point1", "point2"],
  "missed_points": ["point1", "point2"],
  "feedback": "<2-3 sentence encouraging feedback>"
}}"""

    response = bedrock.invoke_model(
        modelId='anthropic.claude-3-haiku-20240307-v1:0',
        contentType='application/json',
        accept='application/json',
        body=json.dumps({
            "anthropic_version": "bedrock-2023-05-31",
            "max_tokens": 1024,
            "messages": [
                {"role": "user", "content": prompt}
            ]
        })
    )

    result = json.loads(response['body'].read())
    return json.loads(result['content'][0]['text'])
```

### Step 3: Complete Evaluate-Response Lambda Handler

```python
import boto3
import json

def lambda_handler(event, context):
    """
    This Lambda is called by Step Functions after transcription completes.
    It orchestrates the full evaluation: KB retrieval + Claude comparison.
    """
    
    # Extract data from Step Functions input
    response_id = event['responseId']
    question_id = event['questionId']
    question_text = event['questionText']
    transcript = event['transcript']
    
    # STEP A: Get correct answer from Knowledge Base
    reference_answer = retrieve_reference(question_text)
    
    # STEP B: Send to Claude for evaluation
    evaluation = evaluate_response(transcript, reference_answer, question_text)
    
    # Return result to Step Functions (next step: StoreResults)
    return {
        'responseId': response_id,
        'questionId': question_id,
        'patientId': event['patientId'],
        'assignmentId': event['assignmentId'],
        'score': evaluation['score'],
        'totalPoints': evaluation['total_points'],
        'percentage': evaluation['percentage'],
        'correctPoints': evaluation['correct_points'],
        'missedPoints': evaluation['missed_points'],
        'feedback': evaluation['feedback'],
        'transcript': transcript,
        'referenceUsed': reference_answer[:200]  # Store snippet for audit
    }
```

---

## Knowledge Base Document Formats

### Reference Answer Document (upload per question)

```json
{
  "question_id": "Q001",
  "question": "What should you avoid in the first 24 hours after a tooth extraction?",
  "category": "post-extraction-care",
  "expected_answer": "Patients should avoid hot foods and beverages, hard or crunchy foods, using straws (suction can dislodge the blood clot), smoking, forceful spitting, and alcohol for at least 24 hours after extraction.",
  "key_points": [
    "Avoid hot foods and beverages",
    "No hard or crunchy foods",
    "Do not use straws",
    "No smoking",
    "Avoid forceful spitting",
    "No alcohol"
  ],
  "scoring_notes": "Each key point = 1 point. Partial credit for related concepts.",
  "total_points": 6,
  "passing_score": 4
}
```

### Scoring Rubric Document (upload per category)

```json
{
  "category": "post-extraction-care",
  "scoring_guidelines": {
    "excellent": "5-6 points — comprehensive understanding",
    "good": "3-4 points — knows basics, missing some details",
    "needs_improvement": "1-2 points — significant knowledge gaps",
    "critical_miss": "0 points — flag for dentist follow-up"
  },
  "flag_for_review_if": "Score is 0-1 or patient mentions harmful practices"
}
```

---

## Bedrock Knowledge Base Configuration

```
Service: Amazon Bedrock Knowledge Bases
Data Source: S3 bucket (dental-ai-knowledge-{env}/)
Embedding Model: Amazon Titan Embeddings G1 - Text v2
Vector Store: Managed OpenSearch Serverless (auto-provisioned by Bedrock)
Chunking Strategy: Fixed-size, 300 tokens, 20% overlap
Sync: On-demand (triggered when dentist uploads new documents)
```

---


---

# PART 3: IMPLEMENTATION PLAN

---

## Phase 1: Foundation (Week 1-2)

| Task | Details |
|------|---------|
| 1.1 | Set up AWS account, IAM roles, organizational structure |
| 1.2 | Create infrastructure as code (AWS CDK TypeScript) |
| 1.3 | Set up Cognito user pools (dentist + patient) |
| 1.4 | Deploy API Gateway with Lambda integrations (stubs) |
| 1.5 | Create DynamoDB tables with GSIs |
| 1.6 | Create S3 buckets with encryption + lifecycle policies |
| 1.7 | Set up CI/CD pipeline (GitHub Actions → CDK Deploy) |

## Phase 2: Admin Backend (Week 2-3)

| Task | Details |
|------|---------|
| 2.1 | Implement question CRUD Lambda functions |
| 2.2 | Implement assignment creation and management |
| 2.3 | Implement KB document upload + sync trigger |
| 2.4 | Set up Bedrock Knowledge Base with initial documents |
| 2.5 | Implement results dashboard API |
| 2.6 | Build dentist admin web app (React/Next.js) |

## Phase 3: Patient Flow (Week 3-4)

| Task | Details |
|------|---------|
| 3.1 | Implement presigned URL generation for uploads |
| 3.2 | Implement response submission handler |
| 3.3 | Build patient app with audio/video recording |
| 3.4 | Implement S3 event → EventBridge → Step Functions trigger |
| 3.5 | Build patient results view |

## Phase 4: AI Processing Pipeline (Week 4-5)

| Task | Details |
|------|---------|
| 4.1 | Implement validate-input Lambda |
| 4.2 | Implement start-transcription Lambda (Amazon Transcribe) |
| 4.3 | Implement check-transcription Lambda (polling) |
| 4.4 | Implement evaluate-response Lambda (Bedrock KB + Claude) |
| 4.5 | Implement store-results Lambda |
| 4.6 | Implement send-notifications Lambda (SNS + SES) |
| 4.7 | Deploy Step Functions state machine |
| 4.8 | End-to-end pipeline testing |

## Phase 5: Polish & Security (Week 5-6)

| Task | Details |
|------|---------|
| 5.1 | Set up WAF rules on API Gateway |
| 5.2 | Configure CloudWatch dashboards and alarms |
| 5.3 | Implement error handling and retry logic |
| 5.4 | Add custom vocabulary to Transcribe (dental terms) |
| 5.5 | Performance testing (concurrent uploads) |
| 5.6 | HIPAA compliance review |
| 5.7 | User acceptance testing |

---

# PART 4: SECURITY, COST & MONITORING

---

## Security Design

### Encryption
- **At rest**: All S3, DynamoDB encrypted with AWS KMS
- **In transit**: TLS 1.2+ on all endpoints
- **Audio files**: Server-side encryption (SSE-KMS)

### Access Control
```
Dentist:
  ✓ Read/Write own questions
  ✓ Manage own patient assignments
  ✓ View own patients' results
  ✓ Upload KB documents

Patient:
  ✓ View own assigned questions
  ✓ Upload responses to own assignments
  ✓ View own results
  ✗ Cannot access other patients' data
```

### HIPAA Compliance
- CloudTrail audit logging enabled
- S3 versioning + lifecycle policies
- DynamoDB point-in-time recovery
- VPC endpoints (no public internet for data)
- BAA (Business Associate Agreement) with AWS

---

## Cost Estimate (1,000 patients/month, 3 questions each = 3,000 evaluations)

| Service | Usage | Monthly Cost |
|---------|-------|-------------|
| Lambda | ~50K invocations | $2 |
| API Gateway | ~100K requests | $0.35 |
| S3 | 50 GB storage | $5 |
| DynamoDB | ~200K reads/writes | $3 |
| Transcribe | 1,500 min (30s avg × 3000) | $36 |
| Bedrock Claude Haiku | 3,000 evaluations | $15 |
| Bedrock Knowledge Bases | 3,000 queries | $30 |
| Step Functions | 3,000 × 7 states | $0.50 |
| Cognito | 1,050 users | Free |
| SNS + SES | 6,000 notifications | $1 |
| CloudFront | Moderate traffic | $3 |
| CloudWatch | Standard monitoring | $5 |
| WAF | Basic rules | $6 |
| KMS | 2 keys | $2 |
| **TOTAL** | | **~$109/month** |

---

## Error Handling & Retry Strategy

| Scenario | Handling |
|----------|----------|
| Audio upload fails | Client retries with exponential backoff |
| Transcribe job fails | Step Functions retries 3× with 60s delay |
| Bedrock evaluation fails | Retry 2×; if still failing → "needs-manual-review" |
| DynamoDB write fails | Built-in SDK retry |
| Notification fails | Dead letter queue (SQS DLQ) |
| Invalid file format | Immediate rejection; patient notified |
| KB has no matching answer | Fallback to general evaluation; flag for dentist |

---

## Monitoring & Alerting

| Metric | Alarm Threshold | Action |
|--------|----------------|--------|
| Lambda errors | > 5% error rate | Page on-call |
| Transcribe duration | > 5 minutes | Warning alert |
| Step Functions failures | Any failure | Email dev team |
| API Gateway 5xx | > 1% | Page on-call |
| Bedrock throttling | Any throttle | Queue or alert |

---

## Technology Stack

| Layer | Technology |
|-------|-----------|
| Frontend (Admin) | React / Next.js |
| Frontend (Patient) | React Native or React Web |
| Backend | Python 3.12 (Lambda) |
| Infrastructure as Code | AWS CDK (TypeScript) |
| Database | DynamoDB |
| File Storage | S3 |
| AI - Speech | Amazon Transcribe |
| AI - Evaluation | Amazon Bedrock (Claude 3 Haiku) |
| AI - RAG | Bedrock Knowledge Bases |
| Orchestration | AWS Step Functions |
| Auth | Amazon Cognito |
| CI/CD | GitHub Actions |
| Monitoring | CloudWatch + X-Ray |

---

## Sequence Diagram — Full Flow

```
Patient        App        API GW     Lambda      S3       EventBridge  StepFn    Transcribe  Bedrock KB  Bedrock AI  DynamoDB   SNS/SES
  │             │           │          │         │            │          │           │           │          │          │          │
  │─Record──────▶           │          │         │            │          │           │           │          │          │          │
  │             │─GET URL───▶──────────▶         │            │          │           │           │          │          │          │
  │             │◀──presigned URL───────┤         │            │          │           │           │          │          │          │
  │             │─Upload audio─────────────────▶│            │          │           │           │          │          │          │
  │             │─POST /responses──▶───────────▶│            │          │           │           │          │          │          │
  │             │           │          │─metadata─────────────────────────────────────────────────────────▶│          │          │
  │             │           │          │         │─S3 event──▶│          │           │           │          │          │          │
  │             │           │          │         │            │─Start───▶│           │           │          │          │          │
  │             │           │          │         │            │          │─Transcribe▶           │          │          │          │
  │             │           │          │         │            │          │◀─Text─────┤           │          │          │          │
  │             │           │          │         │            │          │─Get ref answer────────▶           │          │          │
  │             │           │          │         │            │          │◀─Reference─────────────┤          │          │          │
  │             │           │          │         │            │          │─Evaluate──────────────────────────▶          │          │
  │             │           │          │         │            │          │◀─Score + Feedback─────────────────┤          │          │
  │             │           │          │         │            │          │─Store──────────────────────────────────────▶│          │
  │             │           │          │         │            │          │─Notify───────────────────────────────────────────────▶│
  │◀─────────────Push notification "Results ready"──────────────────────────────────────────────────────────────────────────────┤
  │─View────────▶─GET /results─▶───────▶─Query───────────────────────────────────────────────────────────▶│          │          │
  │◀─────────────Results + Feedback────┤         │            │          │           │           │          │          │          │
```

---

## Future Enhancements

| Enhancement | Description |
|-------------|-------------|
| Multi-language | Transcribe supports 100+ languages |
| Video analysis | Rekognition for visual responses |
| Adaptive questions | AI suggests follow-up questions |
| Progress tracking | Patient improvement over time |
| Bulk assignments | Question sets to patient groups |
| PDF reports | Export patient performance |
| Real-time streaming | Live audio evaluation |
| Custom models | Fine-tune on dentist feedback |
