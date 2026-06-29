# Dental Trainee AI Evaluation System — Complete Design Document

---

# PART 1: WHAT IS THIS SYSTEM?

---

## Business Scenario

```
WHO YOU ARE:      A dental training company/institute
WHAT YOU DO:      Train junior/trainee dentists
THE PROBLEM:      You need to evaluate if trainees can handle real patients
THE SOLUTION:     AI watches/listens to their responses and evaluates them
HOW IT WORKS:     You send clinical scenarios (text + images)
                  Trainees respond via video or audio
                  AI evaluates on 8 dimensions
```

---

## How It Works (Simple Version)

```
STEP 1: Admin creates clinical scenario (text + clinical images + correct approach)
STEP 2: Trainee receives scenario on their app
STEP 3: Trainee records a VIDEO or AUDIO response (like a viva/interview)
STEP 4: AI watches/listens and evaluates on 8 dimensions
STEP 5: Trainee and Admin both see detailed results + feedback
```

---

## Real Example — One Complete Cycle

### Clinical Scenario (created by senior dentist):

> **Case:** A 45 year old patient Mrs Mariya has come back 2 days after a silver
> filling with a complaint of pain. The pain is present in her lower left second
> molar. The pain is mainly on biting and started soon after the silver filling.
> The cavity was shallow. The patient is worried and annoyed at the earlier dentist
> and wants an explanation. She has tried painkillers but nothing helps.
>
> **Clinical Image:** [Photo of molar with silver amalgam filling]
>
> **Task:** You being her doctor for today, need to take a history from the patient,
> help her with a diagnosis and management. Answer patient questions.

### Correct Approach (uploaded to Knowledge Base by admin):

> - Take history: pain on biting (occlusal), timing (started after filling),
>   type (sharp), hot/cold sensitivity, any clenching habits
> - Likely diagnosis: Occlusal high point on restoration (most common cause of
>   post-filling pain on biting), or galvanic reaction, or reversible pulpitis
> - Management: Check occlusion with articulating paper, adjust filling height,
>   smooth contacts, review in 1 week
> - If persists: consider reversible pulpitis, sedative dressing, or replacement
> - Patient communication: Explain simply that the filling might be slightly too
>   high, reassure it's a quick fix, acknowledge frustration with empathy

### Trainee Records a 3-Minute Video Response

### AI Evaluation Result:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                                                                          │
│  📋 HISTORY TAKING: 7/10                                                 │
│     "Asked about pain type and timing. Missed: asking about hot/cold     │
│      sensitivity, previous dental history, clenching/grinding habits."   │
│                                                                          │
│  🧠 CLINICAL REASONING: 8/10                                             │
│     "Correctly linked post-filling pain on biting to occlusal high       │
│      point. Good logical flow from symptoms to diagnosis."               │
│                                                                          │
│  🎯 DIAGNOSIS: 9/10                                                      │
│     "Correctly identified high restoration as primary cause.             │
│      Mentioned possibility of reversible pulpitis."                      │
│                                                                          │
│  💊 MANAGEMENT PLAN: 7/10                                                │
│     "Mentioned occlusal adjustment and review. Missed: checking for      │
│      galvanic reaction, considering temporary sedative dressing."         │
│                                                                          │
│  🗣️ PATIENT COMMUNICATION: 6/10                                          │
│     "Used some technical jargon patient might not understand.            │
│      Could simplify the explanation of 'occlusal high point'."           │
│                                                                          │
│  ❤️ EMPATHY: 8/10                                                        │
│     "Acknowledged patient's frustration. Reassured that the issue        │
│      is fixable. Good rapport building."                                 │
│                                                                          │
│  ⚡ CONFIDENCE: 5/10                                                      │
│     "Hesitated before giving diagnosis. Said 'I think maybe' instead     │
│      of being definitive. Long pause before management plan."            │
│                                                                          │
│  👁️ BODY LANGUAGE: 6/10 (only if video submitted)                        │
│     "Maintained some eye contact but looked away during key              │
│      explanations. Slight fidgeting."                                    │
│                                                                          │
│  📊 OVERALL: 70/100                                                      │
│                                                                          │
│  💬 FEEDBACK:                                                             │
│     "Strong diagnostic skills! Your clinical reasoning is solid.         │
│      Focus on: (1) asking more history questions before jumping to       │
│      diagnosis, (2) being more definitive when explaining — patients     │
│      need to feel you're confident, (3) simplify technical terms         │
│      when speaking to patients. Try: 'Your filling is sitting a bit      │
│      too high, which is why it hurts when you bite. I can fix that       │
│      right now in a few minutes.'"                                       │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Audio vs Video — Both Supported

```
IF trainee uploads VIDEO (.mp4, .webm, .mov):
  → AI evaluates ALL 8 dimensions

IF trainee uploads AUDIO ONLY (.mp3, .wav, .m4a):
  → AI evaluates 7 dimensions (skips body language)
```

| Dimension | Video | Audio-Only |
|-----------|-------|------------|
| History Taking | ✅ | ✅ |
| Clinical Reasoning | ✅ | ✅ |
| Diagnosis | ✅ | ✅ |
| Management Plan | ✅ | ✅ |
| Patient Communication | ✅ | ✅ |
| Empathy | ✅ | ✅ |
| Confidence | ✅ | ✅ |
| Body Language | ✅ | ⏭️ Skipped |

---

## Scenarios Can Include

| Component | Required? | Examples |
|-----------|-----------|---------|
| Case description (text) | Yes | Patient history, symptoms, context |
| Clinical images | Optional | X-rays, intraoral photos, models |
| Task instructions | Yes | "Diagnose and create management plan" |
| Multiple images | Optional | Before/after, multiple views |

The system supports any number of scenarios with any combination of text + images.

---

# PART 2: HOW THE AI EVALUATION WORKS (Detailed)

---

## The Core: What Does AI Receive and How Does It Evaluate?

Claude AI on Amazon Bedrock is **multimodal** — it can understand text, audio, and images all at once.

For each trainee submission, AI receives everything in ONE call:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                                                                          │
│  WHAT AI RECEIVES (in one single API call):                              │
│                                                                          │
│  ┌───────────────────────────────────────────────────────────────────┐  │
│  │  1. SCENARIO TEXT                                                  │  │
│  │     "A 45 year old patient Mrs Mariya..."                         │  │
│  │                                                                    │  │
│  │  2. SCENARIO IMAGES (clinical photos, X-rays)                     │  │
│  │     [photo of molar with silver filling]                          │  │
│  │     So AI knows exactly what the trainee is looking at            │  │
│  │                                                                    │  │
│  │  3. CORRECT APPROACH (from Knowledge Base)                        │  │
│  │     "Expected: take history, diagnose high filling, adjust..."    │  │
│  │     This is what AI compares the trainee's answer AGAINST         │  │
│  │                                                                    │  │
│  │  4. TRAINEE'S AUDIO (their spoken response)                       │  │
│  │     AI can hear: words + tone + hesitations + pauses + filler     │  │
│  │                                                                    │  │
│  │  5. TRAINEE'S VIDEO FRAMES (if video submitted)                   │  │
│  │     AI can see: body language + eye contact + posture             │  │
│  │     Skipped if audio-only submission                              │  │
│  │                                                                    │  │
│  │  6. EVALUATION INSTRUCTIONS (our scoring prompt)                  │  │
│  │     "Score on 8 dimensions: history taking, reasoning..."         │  │
│  └───────────────────────────────────────────────────────────────────┘  │
│                                                                          │
│  WHAT AI DOES:                                                           │
│                                                                          │
│  • Listens to audio → understands what trainee said + how they said it  │
│  • Looks at scenario image → understands the clinical context           │
│  • Compares trainee's response against correct approach                 │
│  • Scores each dimension separately                                      │
│  • Generates personalized feedback with improvement tips                 │
│                                                                          │
│  WHAT AI RETURNS:                                                        │
│                                                                          │
│  Structured JSON with 8 scores + overall percentage + feedback           │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Step-by-Step Processing Pipeline

```
TRAINEE UPLOADS VIDEO OR AUDIO
        │
        ▼
┌─────────────────────────────────────────────────────────────────────────┐
│  STEP 1: DETECT FILE TYPE                                                │
│                                                                          │
│  Lambda checks the uploaded file:                                        │
│  • Video (.mp4, .webm, .mov) → process as video (extract audio+frames) │
│  • Audio (.mp3, .wav, .m4a) → process as audio (use directly)          │
└───────────────────────────────────┬─────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│  STEP 2: PROCESS MEDIA                                                   │
│                                                                          │
│  IF VIDEO:                                                               │
│    • Extract audio track (using FFmpeg)                                  │
│    • Extract key frames every 10-15 seconds (using FFmpeg)              │
│    • Example: 3-min video → audio.mp3 + ~12 JPEG frames                │
│                                                                          │
│  IF AUDIO:                                                               │
│    • Use audio file directly                                            │
│    • No frames to extract                                               │
│                                                                          │
│  Tool: AWS Lambda with FFmpeg layer                                      │
└───────────────────────────────────┬─────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│  STEP 3: FETCH SCENARIO DATA                                             │
│                                                                          │
│  From DynamoDB + S3:                                                     │
│    • Scenario text (the clinical case description)                       │
│    • Scenario images (clinical photos, X-rays)                          │
│                                                                          │
│  From Bedrock Knowledge Base:                                            │
│    • Correct approach / expected answer for this scenario               │
│    • Key points that should be covered                                   │
│    • Scoring criteria                                                    │
│                                                                          │
│  WHY: AI needs the full context to evaluate properly                     │
└───────────────────────────────────┬─────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│  STEP 4: AI EVALUATION (ONE API CALL TO CLAUDE)                          │
│                                                                          │
│  Send to Amazon Bedrock (Claude model):                                  │
│    • Scenario text + images                                             │
│    • Correct approach from KB                                           │
│    • Trainee's audio                                                    │
│    • Trainee's video frames (if video)                                  │
│    • Scoring prompt with 8 dimensions                                   │
│                                                                          │
│  Claude processes everything and returns:                                │
│    • 8 individual scores with explanations                              │
│    • Overall percentage                                                  │
│    • Personalized feedback with specific tips                           │
└───────────────────────────────────┬─────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│  STEP 5: STORE RESULTS + NOTIFY                                          │
│                                                                          │
│  • Save full evaluation to DynamoDB                                      │
│  • Send push notification to trainee: "Results ready"                    │
│  • Send email summary to admin                                           │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## The 8 Evaluation Dimensions Explained

| # | Dimension | What AI Checks | How AI Detects It |
|---|-----------|---------------|-------------------|
| 1 | **History Taking** | Did trainee ask the right questions before diagnosing? | Listens for: questions about pain type, duration, triggers, medical history |
| 2 | **Clinical Reasoning** | Did they logically connect symptoms to diagnosis? | Listens for: "because X, therefore Y" type reasoning |
| 3 | **Diagnosis** | Is the final diagnosis correct? | Compares stated diagnosis against correct answer from KB |
| 4 | **Management Plan** | Are the treatment steps correct and complete? | Compares proposed treatment against expected approach |
| 5 | **Patient Communication** | Is the explanation clear for a non-medical person? | Checks for jargon vs simple language |
| 6 | **Empathy** | Did they acknowledge the patient's feelings/concerns? | Listens for: acknowledgment, reassurance, caring tone |
| 7 | **Confidence** | How confidently did they deliver the answer? | Detects: filler words (um, uh), hesitation, pauses, questioning tone |
| 8 | **Body Language** | Professional visual presence? (video only) | Looks at frames: eye contact, posture, gestures, composure |

---

## What is the Knowledge Base? (RAG)

```
┌─────────────────────────────────────────────────────────────────────────┐
│                                                                          │
│  PROBLEM:                                                                │
│  You don't want AI to use its general knowledge to evaluate.             │
│  You want AI to use YOUR specific correct approaches and criteria.       │
│                                                                          │
│  SOLUTION: Knowledge Base                                                │
│  Upload your correct approaches for each scenario.                       │
│  AI searches HERE first to find what the correct answer should be.       │
│                                                                          │
│  WHAT YOU UPLOAD:                                                        │
│  • Correct approach for each clinical scenario                           │
│  • Key points that must be covered                                       │
│  • Common mistakes to flag                                               │
│  • Scoring rubrics per category                                          │
│  • Clinical guidelines and protocols                                     │
│                                                                          │
│  ANALOGY: It's like giving an answer sheet to the AI examiner.           │
│  The examiner (AI) looks at the answer sheet (Knowledge Base)            │
│  and compares it with what the student (trainee) said.                   │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## What is Amazon Bedrock?

```
┌─────────────────────────────────────────────────────────────────────────┐
│                                                                          │
│  Amazon Bedrock = ONE AWS service with TWO capabilities you'll use:      │
│                                                                          │
│  ┌────────────────────────────────────────────────────────────────┐     │
│  │  1. KNOWLEDGE BASE                                              │     │
│  │     = Stores your correct answers and guidelines                │     │
│  │     = You upload documents, it indexes them automatically       │     │
│  │     = When you query it, it finds the best matching answer      │     │
│  └────────────────────────────────────────────────────────────────┘     │
│                                                                          │
│  ┌────────────────────────────────────────────────────────────────┐     │
│  │  2. CLAUDE MODEL (the AI brain)                                 │     │
│  │     = Can understand text + audio + images (multimodal)         │     │
│  │     = You send it the trainee's recording + correct answer      │     │
│  │     = It evaluates and returns scores + feedback                │     │
│  └────────────────────────────────────────────────────────────────┘     │
│                                                                          │
│  Both are accessed through the SAME service (Bedrock).                  │
│  No separate services needed.                                            │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

# PART 3: COMPLETE TECHNICAL ARCHITECTURE

---

## All Users and Their Actions

```
┌─────────────────────────────────────────────────────────────────────────┐
│  ADMIN / SENIOR DENTIST                                                  │
│                                                                          │
│  • Create clinical scenarios (text + images)                             │
│  • Upload correct approaches to Knowledge Base                           │
│  • Assign scenarios to trainees                                          │
│  • View all trainee results on dashboard                                 │
│  • Track trainee progress over time                                      │
│  • Export reports                                                         │
└─────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────┐
│  TRAINEE DENTIST                                                         │
│                                                                          │
│  • Receive notification: "You have a new assessment"                     │
│  • Open app → see clinical scenario (text + images)                      │
│  • Record VIDEO or AUDIO response                                        │
│  • Submit recording                                                      │
│  • View results + AI feedback                                            │
│  • Track own improvement over time                                       │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Complete Architecture Diagram

```
┌────────────────────────┐                         ┌────────────────────────┐
│   ADMIN WEB APP        │                         │   TRAINEE APP          │
│   (React/Next.js)      │                         │   (React Native/Web)   │
│                        │                         │                        │
│  • Create scenarios    │                         │  • View scenarios      │
│  • Upload images       │                         │  • Record video/audio  │
│  • Upload KB docs      │                         │  • View results        │
│  • View dashboard      │                         │                        │
└───────────┬────────────┘                         └───────────┬────────────┘
            │                                                  │
            ▼                                                  ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         AWS CloudFront (CDN)                                 │
│                    Hosts web apps, HTTPS, fast delivery                      │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         Amazon Cognito (Authentication)                      │
│                    Admin + Trainee login, JWT tokens                         │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         Amazon API Gateway (REST API)                        │
│                                                                             │
│  Admin APIs:                           Trainee APIs:                         │
│  POST /scenarios                       GET  /my/assignments                 │
│  POST /scenarios/{id}/images           GET  /my/scenarios/{assignmentId}    │
│  POST /knowledge-base/upload           POST /my/responses/upload-url        │
│  POST /assignments                     POST /my/responses                   │
│  GET  /results                         GET  /my/results                     │
│  GET  /dashboard/stats                                                      │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         AWS Lambda (Backend Logic)                           │
│                                                                             │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐     │
│  │  Scenario    │ │  Assignment  │ │  Upload URL  │ │  Results     │     │
│  │  Manager    │ │  Manager     │ │  Generator   │ │  Fetcher     │     │
│  └──────────────┘ └──────────────┘ └──────────────┘ └──────────────┘     │
│                                                                             │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐     │
│  │  File Type   │ │  Media       │ │  Evaluate    │ │  Store &     │     │
│  │  Detector   │ │  Processor   │ │  Response    │ │  Notify      │     │
│  └──────────────┘ └──────────────┘ └──────────────┘ └──────────────┘     │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
              ┌─────────────────────┼─────────────────────┐
              ▼                     ▼                     ▼
┌──────────────────┐   ┌──────────────────┐   ┌──────────────────────────┐
│  Amazon S3       │   │  Amazon DynamoDB │   │  AWS Step Functions      │
│                  │   │                  │   │  (Processing Pipeline)    │
│  • Videos/audio  │   │  • Scenarios     │   │                          │
│  • Scenario imgs │   │  • Assignments   │   │  Detect → Process →      │
│  • KB documents  │   │  • Results       │   │  Fetch → Evaluate →      │
│  • Extracted     │   │  • Users         │   │  Store → Notify          │
│    frames        │   │                  │   │                          │
└──────────────────┘   └──────────────────┘   └────────────┬─────────────┘
                                                           │
                                          ┌────────────────┼────────────┐
                                          ▼                ▼            ▼
                                ┌──────────────┐ ┌──────────────┐ ┌─────────┐
                                │  Bedrock     │ │  Bedrock     │ │ SNS+SES │
                                │  Knowledge   │ │  Claude AI   │ │ Notify  │
                                │  Base        │ │  (Evaluator) │ │         │
                                │              │ │              │ │ Push +  │
                                │  Your correct│ │  Listens,    │ │ Email   │
                                │  answers     │ │  sees, and   │ │         │
                                │  stored here │ │  evaluates   │ │         │
                                └──────────────┘ └──────────────┘ └─────────┘
```

---

## Processing Pipeline Detail

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                AWS STEP FUNCTIONS — Evaluation Pipeline                       │
│                                                                              │
│  Triggered automatically when trainee uploads file to S3                     │
│                                                                              │
│  ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐│
│  │  STEP 1  │──▶│  STEP 2  │──▶│  STEP 3  │──▶│  STEP 4  │──▶│  STEP 5  ││
│  │          │   │          │   │          │   │          │   │          ││
│  │ Detect & │   │ Process  │   │  Fetch   │   │    AI    │   │ Store +  ││
│  │ Validate │   │  Media   │   │ Scenario │   │ Evaluate │   │  Notify  ││
│  │          │   │          │   │ + Ref Ans│   │          │   │          ││
│  └──────────┘   └──────────┘   └──────────┘   └──────────┘   └──────────┘│
│       │              │              │               │              │        │
│       ▼              ▼              ▼               ▼              ▼        │
│  • Check file   • If VIDEO:    • Get scenario   • Send ALL to  • Save to  │
│    type/size      extract        text + images    Claude AI:     DynamoDB  │
│  • Is valid?      audio +      • Query KB for    - Scenario     • Send     │
│                   frames         correct          - Images       push to   │
│               • If AUDIO:        approach         - Audio        trainee   │
│                 use directly                      - Frames       • Email    │
│                                                  - Instructions  admin     │
│                                                                            │
│                                                  Returns:                   │
│                                                  8 scores +                 │
│                                                  feedback                   │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Sequence Diagram — Full Flow

```
Trainee      App       API GW    Lambda      S3       StepFn     Bedrock KB   Claude AI    DynamoDB   SNS/SES
  │           │          │         │          │          │            │           │           │          │
  │─Open app──▶          │         │          │          │            │           │           │          │
  │           │─GET scenarios──────▶│          │          │            │           │           │          │
  │           │◀─scenario+images───┤          │          │            │           │           │          │
  │           │          │         │          │          │            │           │           │          │
  │─Record────▶          │         │          │          │            │           │           │          │
  │           │─GET upload URL─────▶│          │          │            │           │           │          │
  │           │◀─presigned URL─────┤          │          │            │           │           │          │
  │           │─Upload file directly─────────▶│          │            │           │           │          │
  │           │─POST /responses────▶│          │          │            │           │           │          │
  │           │          │         │          │          │            │           │           │          │
  │           │          │         │          │─Event───▶│            │           │           │          │
  │           │          │         │          │          │─Detect type│           │           │          │
  │           │          │         │          │          │─Process media          │           │          │
  │           │          │         │          │          │─Get ref answer────────▶│           │          │
  │           │          │         │          │          │◀─Correct approach──────┤           │          │
  │           │          │         │          │          │─Evaluate (all inputs)──────────────▶          │
  │           │          │         │          │          │◀─8 scores + feedback───────────────┤          │
  │           │          │         │          │          │─Store results──────────────────────────────▶│  │
  │           │          │         │          │          │─Notify─────────────────────────────────────────▶
  │           │          │         │          │          │            │           │           │          │
  │◀───────────Push: "Results ready"─────────────────────────────────────────────────────────────────────┤
  │           │          │         │          │          │            │           │           │          │
  │─View──────▶─GET results────────▶│─Query──────────────────────────────────────────────────▶│          │
  │◀──────────Scores + Feedback────┤          │          │            │           │           │          │
```

---

## AWS Services — What Each Does

| Service | What It Does | Why You Need It |
|---------|-------------|-----------------|
| **CloudFront** | Hosts web apps (admin + trainee) | Fast loading globally |
| **Cognito** | Login system (signup, login, OTP) | Secure authentication |
| **API Gateway** | Front door for all API requests | Routes requests to Lambda |
| **Lambda** | Backend code (runs on demand) | No servers to manage |
| **S3** | File storage (videos, images, docs) | Cheap, unlimited, secure |
| **DynamoDB** | Database (scenarios, results, users) | Fast, auto-scales |
| **Step Functions** | Pipeline manager (step-by-step processing) | Handles retries, errors |
| **EventBridge** | Triggers pipeline when file uploaded | Automatic, no polling |
| **Bedrock KB** | Stores correct answers (RAG) | AI uses YOUR data, not guesses |
| **Bedrock Claude** | AI brain (evaluates recordings) | Understands audio + images |
| **SNS** | Push notifications + SMS | Alert trainee |
| **SES** | Email notifications | Alert admin |
| **WAF** | Security (protects API) | Prevents attacks |
| **KMS** | Encryption | All data encrypted |
| **IAM** | Permissions | Controls access |
| **CloudWatch** | Monitoring + alerts | Track errors, performance |

---

# PART 4: DATABASE & API DESIGN

---

## API Endpoints

### Admin APIs

| Method | Endpoint | What It Does |
|--------|----------|-------------|
| POST | /scenarios | Create clinical scenario (text + images) |
| GET | /scenarios | List all scenarios |
| PUT | /scenarios/{id} | Edit scenario |
| DELETE | /scenarios/{id} | Delete scenario |
| POST | /scenarios/{id}/images | Upload clinical images for scenario |
| POST | /assignments | Assign scenarios to trainee(s) |
| GET | /assignments | List all assignments |
| POST | /knowledge-base/upload | Upload correct approach documents |
| GET | /results | Get all trainee results (filterable) |
| GET | /results/{traineeId} | Specific trainee's history |
| GET | /dashboard/stats | Aggregated statistics |

### Trainee APIs

| Method | Endpoint | What It Does |
|--------|----------|-------------|
| GET | /my/assignments | See assigned scenarios |
| GET | /my/scenarios/{assignmentId} | Get scenario details (text + images) |
| POST | /my/responses/upload-url | Get secure upload URL for recording |
| POST | /my/responses | Confirm upload, trigger processing |
| GET | /my/results | View own results and feedback |
| GET | /my/progress | View improvement over time |

---

## Database Schema (DynamoDB)

### Scenarios Table

```
PK: SCENARIO#<scenarioId>
SK: META

Fields:
- scenarioId (UUID)
- adminId (who created)
- title ("Post-filling pain case")
- caseDescription (full text of the clinical scenario)
- taskInstructions ("Take history, diagnose, manage, answer patient questions")
- imageKeys (list of S3 keys for clinical images)
- category ("restorative", "endodontics", "periodontics", "oral-surgery", etc.)
- difficulty ("easy" | "medium" | "hard")
- createdAt (timestamp)
- updatedAt (timestamp)
```

### Assignments Table

```
PK: ASSIGNMENT#<assignmentId>
SK: TRAINEE#<traineeId>

Fields:
- assignmentId (UUID)
- adminId (who assigned)
- traineeId (who receives)
- scenarioIds (list of scenario IDs)
- status ("sent" | "in-progress" | "completed")
- dueDate (timestamp)
- sentAt (timestamp)
- completedAt (timestamp, optional)

GSI: traineeId-index (PK: traineeId, SK: sentAt)
GSI: adminId-status-index (PK: adminId, SK: status)
```

### Responses Table

```
PK: RESPONSE#<responseId>
SK: SCENARIO#<scenarioId>

Fields:
- responseId (UUID)
- traineeId
- assignmentId
- scenarioId
- s3Key (path to video/audio file)
- fileType ("video/mp4" | "audio/mp3" | etc.)
- inputMode ("video" | "audio")
- duration (seconds)
- status ("uploaded" | "processing" | "evaluating" | "completed" | "failed")
- uploadedAt (timestamp)
- processedAt (timestamp, optional)

GSI: traineeId-index (PK: traineeId, SK: uploadedAt)
GSI: status-index (PK: status, SK: uploadedAt)
```

### Results Table

```
PK: RESULT#<resultId>
SK: TRAINEE#<traineeId>

Fields:
- resultId (UUID)
- responseId
- traineeId
- scenarioId
- assignmentId
- inputMode ("video" | "audio")
- historyTaking (object: {score, total: 10, details})
- clinicalReasoning (object: {score, total: 10, details})
- diagnosis (object: {score, total: 10, details})
- managementPlan (object: {score, total: 10, details})
- patientCommunication (object: {score, total: 10, details})
- empathy (object: {score, total: 10, details})
- confidence (object: {score, total: 10, details})
- bodyLanguage (object: {score, total: 10, details} — null if audio-only)
- overallPercentage (number)
- feedback (string — AI-generated personalized tips)
- evaluatedAt (timestamp)
- modelUsed (string — e.g. "claude-3-5-sonnet")

GSI: traineeId-index (PK: traineeId, SK: evaluatedAt)
GSI: scenarioId-index (PK: scenarioId, SK: evaluatedAt)
```

### Users Table

```
PK: USER#<userId>
SK: PROFILE

Fields:
- userId (UUID)
- email
- name
- role ("admin" | "trainee")
- organizationId
- createdAt (timestamp)
```

---

## S3 Bucket Structure

```
dental-trainee-media-{env}/
├── responses/
│   └── {traineeId}/{responseId}.{mp4|webm|mp3|wav}
├── processed/
│   └── {responseId}/
│       ├── audio.mp3
│       └── frames/
│           ├── frame_001.jpg
│           ├── frame_002.jpg
│           └── ...
└── scenarios/
    └── {scenarioId}/
        ├── image_01.jpg
        ├── image_02.jpg
        └── ...

dental-trainee-knowledge-{env}/
├── correct-approaches/
│   └── {scenarioId}.json
├── scoring-rubrics/
│   └── {category}-rubric.json
└── clinical-guidelines/
    └── {topic}.pdf
```

---

# PART 5: IMPLEMENTATION CODE

---

## Lambda Functions Overview

| # | Function | Trigger | Job |
|---|----------|---------|-----|
| 1 | scenario-manager | API Gateway | CRUD for clinical scenarios |
| 2 | assignment-manager | API Gateway | Assign scenarios to trainees |
| 3 | upload-url-generator | API Gateway | Generate presigned S3 URL |
| 4 | response-handler | API Gateway | Record upload metadata |
| 5 | detect-file-type | Step Functions | Check video vs audio |
| 6 | media-processor | Step Functions | Extract audio + frames (FFmpeg) |
| 7 | fetch-scenario-data | Step Functions | Get scenario text + images from S3 |
| 8 | fetch-reference | Step Functions | Query KB for correct approach |
| 9 | evaluate-response | Step Functions | Send everything to Claude AI |
| 10 | store-results | Step Functions | Save scores to DynamoDB |
| 11 | send-notifications | Step Functions | Push + email notifications |
| 12 | results-fetcher | API Gateway | Query results for dashboard |
| 13 | kb-sync | API Gateway / S3 | Re-index Knowledge Base |

---

## Core Code: evaluate-response Lambda

```python
import boto3
import json
import base64

bedrock = boto3.client('bedrock-runtime')
s3 = boto3.client('s3')

BUCKET = 'dental-trainee-media'

def lambda_handler(event, context):
    """
    THE MAIN EVALUATION STEP.
    Sends scenario + audio + frames + reference to Claude.
    Claude evaluates on 8 dimensions and returns scores.
    """
    
    mode = event['inputMode']  # "video" or "audio"
    scenario_text = event['scenarioText']
    task_instructions = event['taskInstructions']
    reference_answer = event['referenceAnswer']
    audio_s3_key = event['audioS3Key']
    frames_s3_keys = event.get('framesS3Keys', [])
    scenario_image_keys = event.get('scenarioImageKeys', [])
    
    # Build the message content array
    content = []
    
    # 1. Add evaluation instructions
    if mode == "video":
        prompt = build_video_prompt(scenario_text, task_instructions, reference_answer)
    else:
        prompt = build_audio_prompt(scenario_text, task_instructions, reference_answer)
    
    content.append({"type": "text", "text": prompt})
    
    # 2. Add scenario images (clinical photos the trainee was shown)
    for img_key in scenario_image_keys:
        img_data = s3.get_object(Bucket=BUCKET, Key=img_key)
        img_base64 = base64.b64encode(img_data['Body'].read()).decode()
        content.append({
            "type": "image",
            "source": {"type": "base64", "media_type": "image/jpeg", "data": img_base64}
        })
    
    content.append({"type": "text", "text": "Above are the clinical images shown to the trainee. Below is their recorded response:"})
    
    # 3. Add trainee's audio
    audio_data = s3.get_object(Bucket=BUCKET, Key=audio_s3_key)
    audio_base64 = base64.b64encode(audio_data['Body'].read()).decode()
    content.append({
        "type": "audio",
        "source": {"type": "base64", "media_type": "audio/mp3", "data": audio_base64}
    })
    
    # 4. Add trainee's video frames (if video mode)
    if mode == "video" and frames_s3_keys:
        content.append({"type": "text", "text": "Below are video frames from the trainee's recording (for body language assessment):"})
        for frame_key in frames_s3_keys[:10]:
            frame_data = s3.get_object(Bucket=BUCKET, Key=frame_key)
            frame_base64 = base64.b64encode(frame_data['Body'].read()).decode()
            content.append({
                "type": "image",
                "source": {"type": "base64", "media_type": "image/jpeg", "data": frame_base64}
            })
    
    # 5. Call Claude AI
    response = bedrock.invoke_model(
        modelId='anthropic.claude-3-5-sonnet-20241022-v2:0',
        contentType='application/json',
        accept='application/json',
        body=json.dumps({
            "anthropic_version": "bedrock-2023-05-31",
            "max_tokens": 3000,
            "messages": [{"role": "user", "content": content}]
        })
    )
    
    result = json.loads(response['body'].read())
    evaluation = json.loads(result['content'][0]['text'])
    
    return {
        'responseId': event['responseId'],
        'traineeId': event['traineeId'],
        'scenarioId': event['scenarioId'],
        'assignmentId': event['assignmentId'],
        'inputMode': mode,
        'evaluation': evaluation
    }


def build_video_prompt(scenario_text, task_instructions, reference_answer):
    return f"""You are evaluating a TRAINEE DENTIST's video response to a clinical scenario.

CLINICAL SCENARIO (shown to trainee):
{scenario_text}

TASK GIVEN TO TRAINEE:
{task_instructions}

CORRECT/EXPECTED APPROACH (from our knowledge base):
{reference_answer}

EVALUATION INSTRUCTIONS:
Listen to the trainee's audio and look at their video frames.
The clinical images above are what the trainee was shown as part of the scenario.

Score on these 8 dimensions (each 0-10):

1. HISTORY TAKING: Did they ask appropriate questions before diagnosing?
2. CLINICAL REASONING: Did they logically connect findings to diagnosis?
3. DIAGNOSIS: Is their diagnosis correct?
4. MANAGEMENT PLAN: Are treatment steps correct and complete?
5. PATIENT COMMUNICATION: Did they explain clearly in simple language?
6. EMPATHY: Did they acknowledge patient's feelings and concerns?
7. CONFIDENCE: How confidently did they speak? (filler words, hesitation, pauses)
8. BODY LANGUAGE: Eye contact, posture, gestures, professional demeanor.

IMPORTANT RULES:
- If communication is poor but diagnosis is correct, give high diagnosis but low communication.
- Confidence is about HOW they deliver, not WHAT they say.
- Be encouraging — these are learners.
- Give specific, actionable improvement tips.

Respond ONLY in this JSON format:
{{
  "history_taking": {{"score": 0, "total": 10, "details": "..."}},
  "clinical_reasoning": {{"score": 0, "total": 10, "details": "..."}},
  "diagnosis": {{"score": 0, "total": 10, "details": "..."}},
  "management_plan": {{"score": 0, "total": 10, "details": "..."}},
  "patient_communication": {{"score": 0, "total": 10, "details": "..."}},
  "empathy": {{"score": 0, "total": 10, "details": "..."}},
  "confidence": {{"score": 0, "total": 10, "details": "..."}},
  "body_language": {{"score": 0, "total": 10, "details": "..."}},
  "overall_percentage": 0,
  "feedback": "2-3 sentences with specific improvement tips"
}}"""


def build_audio_prompt(scenario_text, task_instructions, reference_answer):
    return f"""You are evaluating a TRAINEE DENTIST's audio response to a clinical scenario.

CLINICAL SCENARIO (shown to trainee):
{scenario_text}

TASK GIVEN TO TRAINEE:
{task_instructions}

CORRECT/EXPECTED APPROACH (from our knowledge base):
{reference_answer}

EVALUATION INSTRUCTIONS:
Listen to the trainee's audio recording.
The clinical images above are what the trainee was shown as part of the scenario.

Score on these 7 dimensions (each 0-10):

1. HISTORY TAKING: Did they ask appropriate questions before diagnosing?
2. CLINICAL REASONING: Did they logically connect findings to diagnosis?
3. DIAGNOSIS: Is their diagnosis correct?
4. MANAGEMENT PLAN: Are treatment steps correct and complete?
5. PATIENT COMMUNICATION: Did they explain clearly in simple language?
6. EMPATHY: Did they acknowledge patient's feelings and concerns?
7. CONFIDENCE: How confidently did they speak? (filler words, hesitation, pauses)

NOTE: Body language cannot be assessed (audio-only submission).

IMPORTANT RULES:
- If communication is poor but diagnosis is correct, give high diagnosis but low communication.
- Confidence is about HOW they deliver, not WHAT they say.
- Be encouraging — these are learners.
- Give specific, actionable improvement tips.

Respond ONLY in this JSON format:
{{
  "history_taking": {{"score": 0, "total": 10, "details": "..."}},
  "clinical_reasoning": {{"score": 0, "total": 10, "details": "..."}},
  "diagnosis": {{"score": 0, "total": 10, "details": "..."}},
  "management_plan": {{"score": 0, "total": 10, "details": "..."}},
  "patient_communication": {{"score": 0, "total": 10, "details": "..."}},
  "empathy": {{"score": 0, "total": 10, "details": "..."}},
  "confidence": {{"score": 0, "total": 10, "details": "..."}},
  "body_language": null,
  "overall_percentage": 0,
  "feedback": "2-3 sentences with specific improvement tips"
}}"""
```

---

## Knowledge Base: What to Upload Per Scenario

```json
{
  "scenario_id": "SCN001",
  "title": "Post-filling pain — silver amalgam",
  "category": "restorative",
  "correct_approach": {
    "history_questions": [
      "When did the pain start? (immediately after or delayed)",
      "Type of pain? (sharp on biting, dull ache, throbbing)",
      "Hot/cold sensitivity?",
      "Pain localized or radiating?",
      "Any clenching or grinding habits?",
      "Previous dental history on this tooth?"
    ],
    "expected_diagnosis": "Occlusal high point on new restoration (most common cause of immediate post-filling pain on biting). Differential: galvanic reaction, reversible pulpitis from filling placement near pulp.",
    "expected_management": [
      "Check occlusion with articulating paper",
      "Identify and adjust high spots",
      "Smooth and polish adjusted areas",
      "Advise patient pain should resolve within days",
      "Review in 1 week",
      "If persists: consider pulp involvement, may need vitality test"
    ],
    "patient_communication_expectations": [
      "Explain in simple terms: filling is sitting too high",
      "Reassure: this is common and easily fixable",
      "Acknowledge frustration",
      "Explain what you will do to fix it"
    ],
    "common_mistakes_to_flag": [
      "Jumping to diagnosis without taking history",
      "Not checking occlusion",
      "Suggesting extraction without conservative options first",
      "Using excessive jargon with worried patient"
    ]
  }
}
```

---

## Step Functions State Machine

```json
{
  "Comment": "Dental Trainee Evaluation Pipeline",
  "StartAt": "DetectFileType",
  "States": {
    "DetectFileType": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:...:detect-file-type",
      "Next": "ProcessMedia",
      "Catch": [{"ErrorEquals": ["States.ALL"], "Next": "MarkFailed"}]
    },
    "ProcessMedia": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:...:media-processor",
      "Next": "FetchScenarioData",
      "TimeoutSeconds": 300,
      "Catch": [{"ErrorEquals": ["States.ALL"], "Next": "MarkFailed"}]
    },
    "FetchScenarioData": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:...:fetch-scenario-data",
      "Next": "FetchReference"
    },
    "FetchReference": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:...:fetch-reference",
      "Next": "EvaluateResponse",
      "Catch": [{"ErrorEquals": ["States.ALL"], "Next": "EvaluateWithoutReference"}]
    },
    "EvaluateResponse": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:...:evaluate-response",
      "Next": "StoreResults",
      "TimeoutSeconds": 120,
      "Retry": [{"ErrorEquals": ["States.ALL"], "MaxAttempts": 2, "BackoffRate": 2}],
      "Catch": [{"ErrorEquals": ["States.ALL"], "Next": "MarkFailed"}]
    },
    "EvaluateWithoutReference": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:...:evaluate-response",
      "Comment": "Fallback: evaluate without KB reference, flag for admin review",
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
      "End": true,
      "Catch": [{"ErrorEquals": ["States.ALL"], "Next": "NotificationFailed"}]
    },
    "NotificationFailed": {
      "Type": "Pass",
      "Comment": "Notification failed but results are saved. Will retry via DLQ.",
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

# PART 6: IMPLEMENTATION PLAN, COST & SECURITY

---

## Implementation Timeline (6 Weeks)

### Phase 1: Foundation (Week 1-2)

| # | Task | Details |
|---|------|---------|
| 1.1 | AWS Account Setup | Create account, billing alerts, organization |
| 1.2 | IAM Roles & Policies | Roles for each Lambda, Step Functions, Bedrock access |
| 1.3 | Infrastructure as Code | AWS CDK (TypeScript) project setup |
| 1.4 | Cognito User Pools | Admin + Trainee authentication |
| 1.5 | API Gateway | Deploy REST API with Lambda stubs |
| 1.6 | DynamoDB Tables | Create all tables with GSIs |
| 1.7 | S3 Buckets | Media bucket + Knowledge bucket with encryption |
| 1.8 | CI/CD Pipeline | GitHub Actions → CDK deploy per environment |

### Phase 2: Admin Features (Week 2-3)

| # | Task | Details |
|---|------|---------|
| 2.1 | Scenario CRUD | Create/edit/delete clinical scenarios |
| 2.2 | Image Upload | Upload clinical images per scenario |
| 2.3 | Assignment Manager | Assign scenarios to trainees |
| 2.4 | KB Document Upload | Upload correct approaches + trigger sync |
| 2.5 | Bedrock KB Setup | Create Knowledge Base, connect S3 source |
| 2.6 | Dashboard API | Results query, filters, aggregation |
| 2.7 | Admin Web App | React/Next.js interface |

### Phase 3: Trainee Features (Week 3-4)

| # | Task | Details |
|---|------|---------|
| 3.1 | Presigned URL Generator | Secure upload URLs for video/audio |
| 3.2 | Response Handler | Record metadata, update status |
| 3.3 | Trainee App | Mobile/web app with scenario display + recording |
| 3.4 | Event Trigger | S3 upload → EventBridge → Step Functions |
| 3.5 | Results View | Scores + feedback display |
| 3.6 | Progress Tracking | Historical scores, improvement graph |

### Phase 4: AI Evaluation Pipeline (Week 4-5)

| # | Task | Details |
|---|------|---------|
| 4.1 | File Type Detector | Lambda to detect video vs audio |
| 4.2 | Media Processor | Lambda with FFmpeg (audio + frame extraction) |
| 4.3 | Scenario Data Fetcher | Get scenario text + images from DB/S3 |
| 4.4 | Reference Fetcher | Query Bedrock KB for correct approach |
| 4.5 | Evaluator | Send to Claude, parse 8-dimension response |
| 4.6 | Results Storage | Save to DynamoDB |
| 4.7 | Notifications | SNS push + SES email |
| 4.8 | Step Functions Deploy | Complete state machine |
| 4.9 | End-to-End Testing | Full flow: upload → evaluate → results |

### Phase 5: Polish & Launch (Week 5-6)

| # | Task | Details |
|---|------|---------|
| 5.1 | WAF Rules | Rate limiting, attack prevention |
| 5.2 | CloudWatch | Dashboards, alarms, error tracking |
| 5.3 | Error Handling | Retries, dead letter queues, graceful failures |
| 5.4 | Performance Testing | Concurrent uploads, response times |
| 5.5 | Security Review | Access control audit, encryption verification |
| 5.6 | User Acceptance Testing | Real trainees test the system |
| 5.7 | Production Deploy | Final deployment + monitoring setup |

---

## Cost Estimate

**Assumptions: 500 trainees/month, 5 scenarios each = 2,500 evaluations/month**
**Average recording: 2-3 minutes**

| Service | Usage | Monthly Cost |
|---------|-------|-------------|
| Lambda | ~50K invocations | ~$3 |
| API Gateway | ~100K requests | ~$0.35 |
| S3 | ~150GB (videos + images + docs) | ~$5 |
| DynamoDB | ~200K reads/writes | ~$3 |
| Bedrock Claude Sonnet | 2,500 evaluations (audio + images) | ~$60-80 |
| Bedrock Knowledge Bases | 2,500 queries | ~$25 |
| Step Functions | 2,500 executions × 7 states | ~$0.45 |
| Cognito | 550 users | Free |
| SNS + SES | 5,000 notifications | ~$1 |
| CloudFront | Web hosting + delivery | ~$3 |
| CloudWatch | Logs + monitoring | ~$5 |
| WAF | API security | ~$6 |
| KMS | 2 encryption keys | ~$2 |
| **TOTAL** | | **~$115-135/month** |

### Cost Reduction Options

| Option | Savings | Trade-off |
|--------|---------|-----------|
| Use Claude Haiku instead of Sonnet | Save ~$40/month | Slightly less nuanced evaluation |
| Process only audio (skip video frames) | Save ~$10/month | No body language scoring |
| Reduce frame count (5 instead of 10) | Save ~$5/month | Less visual data for AI |

---

## Security Design

### Access Control

```
ADMIN:
  ✅ Create/edit/delete scenarios
  ✅ Upload clinical images
  ✅ Upload knowledge base documents
  ✅ Assign scenarios to any trainee
  ✅ View ALL trainees' results
  ✅ View dashboard/analytics
  ❌ Cannot modify trainee accounts directly

TRAINEE:
  ✅ View OWN assigned scenarios only
  ✅ Upload recordings for OWN assignments only
  ✅ View OWN results only
  ❌ Cannot see other trainees' results
  ❌ Cannot access unassigned scenarios
  ❌ Cannot modify scenarios or KB
```

### Data Protection

| What | How | Why |
|------|-----|-----|
| Videos at rest | KMS encryption (SSE-KMS) | Protect recordings |
| Data in transit | TLS 1.2+ (HTTPS) | Secure transmission |
| Database | DynamoDB encryption | Protect scores/data |
| API access | Cognito JWT tokens | Only authenticated users |
| File uploads | Presigned URLs (15 min expiry) | Prevent unauthorized uploads |
| API protection | WAF rules | Block attacks |
| Audit trail | CloudTrail | Track all access |

---

## Error Handling

| What Goes Wrong | What System Does |
|----------------|-----------------|
| Invalid file type uploaded | Reject immediately, notify trainee |
| File too large (>500MB) | Reject at URL generation step |
| FFmpeg extraction fails | Retry once → mark as "processing-error" |
| Knowledge Base has no match | Evaluate without reference, flag for admin review |
| Claude AI fails/timeout | Retry 2 times → mark as "needs-manual-review" |
| Notification fails | Dead letter queue, retry later |
| Step Functions timeout | Mark failed, alert admin |

---

## Monitoring & Alerts

| What to Watch | Alert When | Who Gets Alerted |
|---------------|-----------|-----------------|
| Lambda errors | Error rate > 5% | Dev team |
| Pipeline failures | Any failure | Dev team + admin |
| API 5xx errors | Rate > 1% | Dev team |
| Evaluation time | > 90 seconds | Dev team |
| Upload failures | > 10 in 5 minutes | Dev team |
| Daily cost | Exceeds threshold | Finance/admin |

---

## Technology Stack

| Layer | Technology | Why |
|-------|-----------|-----|
| Frontend (Admin) | React / Next.js | Modern, component-based |
| Frontend (Trainee) | React Native or React Web | Cross-platform |
| Backend | Python 3.12 (Lambda) | Best for AI/Bedrock SDK |
| Infrastructure | AWS CDK (TypeScript) | Repeatable, version-controlled |
| Database | DynamoDB | Serverless, auto-scales |
| File Storage | S3 | Unlimited, cheap |
| AI Evaluation | Bedrock (Claude 3.5 Sonnet) | Multimodal (audio + images) |
| Knowledge Base | Bedrock Knowledge Bases | Managed RAG |
| Pipeline | Step Functions | Visual, retry built-in |
| Auth | Cognito | Managed, free tier |
| Notifications | SNS + SES | Push + email |
| CI/CD | GitHub Actions | Free, flexible |
| Monitoring | CloudWatch + X-Ray | AWS-native |

---

## How to Test Manually on AWS Console

### Quick Test (5 minutes):

```
1. Go to: AWS Console → Amazon Bedrock → Model access
2. Enable: Claude 3.5 Sonnet
3. Go to: Bedrock → Playgrounds → Chat playground
4. Select model: Claude 3.5 Sonnet
5. Upload: a clinical image + an audio recording of someone answering
6. Type: the evaluation prompt (from code above)
7. Click Send → AI returns evaluation with 8 scores
```

### Full Pipeline Test:

```
1. Upload a video to S3 responses/ folder
2. Watch Step Functions console → execution starts automatically
3. See each step turn green (success)
4. Check DynamoDB → results table has scores
5. Check email → notification sent
```

---

## Future Enhancements

| Feature | Description |
|---------|-------------|
| Real-time feedback | Stream audio, get live hints |
| Peer comparison | "You scored higher than 70% of trainees" |
| Progress graphs | Improvement tracking over weeks/months |
| AI follow-ups | "Based on weak areas, try these scenarios next" |
| Multi-language | Support trainees in different languages |
| Custom rubrics | Each admin defines their own scoring weights |
| Cohort management | Group trainees by batch/year |
| PDF certificates | Auto-generate for trainees who pass |
| Video replay + annotations | Admin watches video with AI highlights |
| LMS integration | Connect to existing learning platforms |
| Simulated patient voice | AI plays the patient role for interactive practice |
