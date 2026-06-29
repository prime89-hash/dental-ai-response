# Dental Trainee AI Evaluation System — Complete Design Document

---

# PART 1: WHAT IS THIS SYSTEM?

---

## Business Scenario

```
WHO YOU ARE:    A dental training company/institute
WHAT YOU DO:    Train junior/trainee dentists
THE PROBLEM:    You need to evaluate if trainees can handle real patients
THE SOLUTION:   AI watches their video/audio responses and evaluates them
```

## How It Works (Simple Version)

```
STEP 1: Admin creates clinical scenario questions + uploads correct answers
STEP 2: Trainee receives questions on their app
STEP 3: Trainee records a VIDEO or AUDIO answer (like a viva/interview)
STEP 4: AI watches/listens and evaluates on multiple dimensions
STEP 5: Trainee and Admin both see detailed results + feedback
```

---

## Real Example — One Complete Cycle

### Question (created by senior dentist):

> "A 35-year-old patient comes to you with severe throbbing pain in the lower
> right molar, facial swelling, and mild fever. What is your diagnosis and
> treatment plan? Explain as if you are speaking to the patient."

### Correct Answer (uploaded to Knowledge Base by admin):

> Diagnosis: Periapical abscess.
> Treatment: Prescribe antibiotics (Amoxicillin 500mg), pain relief (Ibuprofen).
> If swelling is fluctuant, perform incision and drainage.
> Long-term: Root canal therapy or extraction.
> Explain to patient in simple empathetic terms.
> Follow-up in 48-72 hours.

### Trainee Records a 2-Minute Video Answer

### AI Evaluation Result:

```
┌─────────────────────────────────────────────────────────────────────┐
│                                                                      │
│  ✅ CORRECTNESS: 8/10                                                │
│     "Correctly identified abscess, mentioned antibiotics and         │
│      root canal. Missed: didn't mention drainage option."            │
│                                                                      │
│  ⚡ CONFIDENCE: 5/10                                                 │
│     "Said 'um' 12 times, long pauses before key answers,            │
│      voice pitch went up when uncertain"                             │
│                                                                      │
│  🗣️ COMMUNICATION: 6/10                                              │
│     "Answer is correct but explanation was jumbled.                  │
│      Started with treatment before giving diagnosis.                 │
│      A real patient would be confused."                              │
│                                                                      │
│  👁️ BODY LANGUAGE: 5/10 (only if video submitted)                   │
│     "Limited eye contact, looking down frequently,                   │
│      fidgeting with pen, slouched posture"                           │
│                                                                      │
│  🎯 COMPLETENESS: 7/10                                              │
│     "Covered diagnosis and treatment. Missed aftercare              │
│      instructions and patient reassurance."                          │
│                                                                      │
│  📊 OVERALL: 62/100                                                  │
│     "Good clinical knowledge! Work on: eye contact,                  │
│      structuring answers (diagnosis → reasoning → plan),             │
│      and reducing filler words."                                     │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Audio vs Video — Both Supported

```
IF trainee uploads VIDEO (.mp4, .webm, .mov):
  → AI evaluates ALL 5 dimensions (correctness + confidence +
    communication + body language + completeness)

IF trainee uploads AUDIO ONLY (.mp3, .wav, .m4a):
  → AI evaluates 4 dimensions (correctness + confidence +
    communication + completeness)
  → Body language is skipped (nothing to see)
```

| Dimension | Video Upload | Audio-Only Upload |
|-----------|-------------|-------------------|
| Correctness | ✅ Scored | ✅ Scored |
| Confidence (voice tone, hesitation) | ✅ Scored | ✅ Scored |
| Communication (clarity, structure) | ✅ Scored | ✅ Scored |
| Completeness (all points covered?) | ✅ Scored | ✅ Scored |
| Body Language (eye contact, posture) | ✅ Scored | ⏭️ Skipped |

---

# PART 2: HOW THE AI EVALUATION WORKS (Detailed)

---

## The Core Question: How Does AI "Watch" a Video and Evaluate?

Amazon Bedrock with Claude AI is a **multimodal model** — meaning it can understand:
- **Text** (the question + correct answer)
- **Audio** (trainee's voice — words, tone, hesitations, pauses)
- **Images** (video frames — body language, eye contact, posture)

You send all of these to Claude in ONE API call, and it returns a structured evaluation.

---

## Step-by-Step Processing Flow

```
TRAINEE UPLOADS VIDEO/AUDIO
        │
        ▼
┌─────────────────────────────────────────────────────────────────┐
│  STEP A: DETECT FILE TYPE                                        │
│                                                                   │
│  Lambda checks: Is it video or audio-only?                        │
│                                                                   │
│  Video (.mp4, .webm, .mov) → go to Step B + Step C              │
│  Audio (.mp3, .wav, .m4a)  → go to Step B only (skip Step C)    │
└───────────────────────────────┬─────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│  STEP B: EXTRACT AUDIO                                           │
│                                                                   │
│  If video: extract audio track from video file using FFmpeg       │
│  If audio: use file directly (nothing to extract)                │
│                                                                   │
│  Tool: AWS Lambda running FFmpeg                                  │
│  Output: Audio file (.mp3 or .wav)                               │
│                                                                   │
│  WHY: So AI can hear what trainee SAID + HOW they said it        │
└───────────────────────────────┬─────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│  STEP C: EXTRACT VIDEO FRAMES (Only if video uploaded)           │
│                                                                   │
│  Take a screenshot every 10-15 seconds from the video            │
│  Example: 2-minute video → ~8-12 frames                          │
│                                                                   │
│  Tool: AWS Lambda running FFmpeg                                  │
│  Output: 8-12 JPEG images                                        │
│                                                                   │
│  WHY: So AI can see body language, eye contact, posture          │
│  NOTE: Skipped entirely if trainee uploaded audio-only           │
└───────────────────────────────┬─────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│  STEP D: FETCH CORRECT ANSWER FROM KNOWLEDGE BASE                │
│                                                                   │
│  System asks Bedrock Knowledge Base:                              │
│  "For this question, what is the expected correct answer?"        │
│                                                                   │
│  Knowledge Base searches through all uploaded dental documents    │
│  and returns the matching reference answer.                       │
│                                                                   │
│  Tool: Amazon Bedrock Knowledge Bases                             │
│  Output: Text of correct answer + key points + scoring criteria  │
│                                                                   │
│  WHY: AI needs something to COMPARE the trainee's answer against │
└───────────────────────────────┬─────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│  STEP E: SEND EVERYTHING TO CLAUDE AI FOR EVALUATION             │
│                                                                   │
│  One single API call to Amazon Bedrock (Claude model) with:      │
│                                                                   │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │  INPUT TO AI:                                              │  │
│  │                                                            │  │
│  │  1. Question text                                          │  │
│  │  2. Correct answer (from Knowledge Base)                   │  │
│  │  3. Audio file (trainee's voice)                           │  │
│  │  4. Video frames (if video was uploaded)                   │  │
│  │  5. Evaluation instructions (our scoring prompt)           │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                   │
│  What Claude AI does internally:                                  │
│                                                                   │
│  • LISTENS to audio → understands what trainee said              │
│  • COMPARES content vs correct answer → scores CORRECTNESS       │
│  • DETECTS "um", "uh", pauses, shaky voice → scores CONFIDENCE  │
│  • CHECKS if explanation is organized → scores COMMUNICATION     │
│  • LOOKS at video frames → scores BODY LANGUAGE (if available)   │
│  • CHECKS if all key points covered → scores COMPLETENESS       │
│                                                                   │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │  OUTPUT FROM AI:                                           │  │
│  │                                                            │  │
│  │  {                                                         │  │
│  │    "correctness": { "score": 8, "total": 10, "details"..} │  │
│  │    "confidence": { "score": 5, "total": 10, "details".. } │  │
│  │    "communication": { "score": 6, "total": 10, "details"} │  │
│  │    "body_language": { "score": 5, "total": 10, "details"} │  │
│  │    "completeness": { "score": 7, "total": 10, "details" } │  │
│  │    "overall_percentage": 62,                               │  │
│  │    "feedback": "Good knowledge! Work on confidence..."     │  │
│  │  }                                                         │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                   │
│  Tool: Amazon Bedrock (Claude 3.5 Sonnet or Claude 4 Sonnet)    │
└───────────────────────────────┬─────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│  STEP F: STORE RESULTS + NOTIFY                                  │
│                                                                   │
│  • Save evaluation results to DynamoDB (database)                │
│  • Send push notification to trainee: "Results ready"            │
│  • Send email summary to admin/senior dentist                    │
│                                                                   │
│  Tools: DynamoDB + SNS + SES                                     │
└─────────────────────────────────────────────────────────────────┘
```

---

## What is the Knowledge Base? (Explained Simply)

```
┌─────────────────────────────────────────────────────────────────────┐
│                                                                      │
│  PROBLEM: If you just ask AI "is this answer correct?" — AI will    │
│  use its general knowledge. But you want it to use YOUR specific    │
│  correct answers and scoring criteria.                               │
│                                                                      │
│  SOLUTION: Upload your correct answers to a "Knowledge Base"         │
│  AI will search YOUR data first, not guess.                          │
│                                                                      │
│  ┌───────────────────────────────────────────────────────────┐      │
│  │                                                            │      │
│  │  WHAT YOU UPLOAD:                                          │      │
│  │                                                            │      │
│  │  • Correct answers for each question                       │      │
│  │  • Key points that must be mentioned                       │      │
│  │  • Scoring criteria (what gets full marks?)                │      │
│  │  • Dental guidelines and protocols                         │      │
│  │  • Any reference material                                  │      │
│  │                                                            │      │
│  └───────────────────────────────────────────────────────────┘      │
│                                                                      │
│  HOW IT WORKS:                                                       │
│                                                                      │
│  1. You upload documents to S3 bucket                                │
│  2. Bedrock Knowledge Base reads them and creates an index           │
│  3. When a trainee submits an answer, system asks KB:                │
│     "What is the correct answer for question X?"                     │
│  4. KB returns the matching reference answer                         │
│  5. AI uses this reference to compare against trainee's response    │
│                                                                      │
│  ANALOGY: Like giving an answer sheet to the AI examiner             │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## What is Amazon Bedrock? (Explained Simply)

```
┌─────────────────────────────────────────────────────────────────────┐
│                                                                      │
│  Amazon Bedrock = AWS service that lets you use AI models            │
│                                                                      │
│  Think of it as: AWS gives you access to smart AI brains.            │
│  You don't build the AI — you just USE it.                           │
│                                                                      │
│  BEDROCK has two things you'll use:                                  │
│                                                                      │
│  ┌──────────────────────────────────────────────────────────┐       │
│  │  1. KNOWLEDGE BASE                                        │       │
│  │     = Storage for your correct answers                    │       │
│  │     = AI searches here to find the right reference        │       │
│  │     = You upload docs, it indexes them automatically      │       │
│  └──────────────────────────────────────────────────────────┘       │
│                                                                      │
│  ┌──────────────────────────────────────────────────────────┐       │
│  │  2. CLAUDE MODEL (the AI brain)                           │       │
│  │     = Made by Anthropic, available through Bedrock        │       │
│  │     = Can understand text + audio + images                │       │
│  │     = You send it data → it returns evaluation            │       │
│  │     = "Multimodal" = understands multiple input types     │       │
│  └──────────────────────────────────────────────────────────┘       │
│                                                                      │
│  You call ONE service (Bedrock) but use TWO capabilities within it  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

# PART 3: COMPLETE END-TO-END ARCHITECTURE

---

## All Users and Their Actions

```
┌─────────────────────────────────────────────────────────────────────┐
│  ADMIN / SENIOR DENTIST                                              │
│                                                                      │
│  What they do:                                                       │
│  • Create clinical scenario questions                                │
│  • Upload correct answers to Knowledge Base                          │
│  • Assign questions to trainees                                      │
│  • View all trainee results on dashboard                             │
│  • Track progress over time                                          │
│  • Export reports                                                     │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│  TRAINEE DENTIST                                                     │
│                                                                      │
│  What they do:                                                       │
│  • Receive notification: "You have a new assessment"                 │
│  • Open app → see clinical scenario question                         │
│  • Record VIDEO or AUDIO answer                                      │
│  • Submit recording                                                  │
│  • View results + AI feedback                                        │
│  • Track own improvement over time                                   │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Complete Technical Architecture

```
┌────────────────────────┐                         ┌────────────────────────┐
│   ADMIN WEB APP        │                         │   TRAINEE APP          │
│   (React/Next.js)      │                         │   (React Native/Web)   │
└───────────┬────────────┘                         └───────────┬────────────┘
            │                                                  │
            ▼                                                  ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         AWS CloudFront (CDN)                                 │
│                    Hosts web apps, fast delivery, HTTPS                      │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         Amazon Cognito (Authentication)                      │
│                    Admin accounts + Trainee accounts + JWT tokens            │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         Amazon API Gateway (REST API)                        │
│                                                                             │
│  Admin APIs:                           Trainee APIs:                         │
│  POST /questions                       GET  /my/assignments                 │
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
│  │  Question    │ │  Assignment  │ │  Upload URL  │ │  Results     │     │
│  │  Manager    │ │  Manager     │ │  Generator   │ │  Fetcher     │     │
│  └──────────────┘ └──────────────┘ └──────────────┘ └──────────────┘     │
│                                                                             │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐     │
│  │  File Type   │ │  Media       │ │  Evaluate    │ │  Store       │     │
│  │  Detector   │ │  Processor   │ │  Response    │ │  Results     │     │
│  │  (audio/vid)│ │  (FFmpeg)    │ │  (Bedrock)   │ │  (DynamoDB)  │     │
│  └──────────────┘ └──────────────┘ └──────────────┘ └──────────────┘     │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
              ┌─────────────────────┼─────────────────────┐
              ▼                     ▼                     ▼
┌──────────────────┐   ┌──────────────────┐   ┌──────────────────────────┐
│  Amazon S3       │   │  Amazon DynamoDB │   │  AWS Step Functions      │
│                  │   │                  │   │  (Processing Pipeline)    │
│  • Video files   │   │  • Questions     │   │                          │
│  • Audio files   │   │  • Assignments   │   │  Detect → Process →      │
│  • KB documents  │   │  • Results       │   │  Evaluate → Store →      │
│  • Video frames  │   │  • Users         │   │  Notify                  │
└──────────────────┘   └──────────────────┘   └────────────┬─────────────┘
                                                           │
                                          ┌────────────────┼────────────────┐
                                          ▼                ▼                ▼
                                ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
                                │  Bedrock     │ │  Bedrock     │ │  SNS + SES   │
                                │  Knowledge   │ │  Claude AI   │ │  (Notify)    │
                                │  Base        │ │              │ │              │
                                │  (Your       │ │  Listens to  │ │  Push + Email│
                                │  correct     │ │  audio, sees │ │  to trainee  │
                                │  answers)    │ │  video, and  │ │  and admin   │
                                │              │ │  evaluates   │ │              │
                                └──────────────┘ └──────────────┘ └──────────────┘
```

---

## Processing Pipeline (What Happens After Upload)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    AWS STEP FUNCTIONS — Processing Pipeline                   │
│                                                                              │
│  Triggered automatically when trainee uploads file to S3                     │
│                                                                              │
│                                                                              │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────┐ │
│  │  STEP 1  │───▶│  STEP 2  │───▶│  STEP 3  │───▶│  STEP 4  │───▶│STEP 5│ │
│  │          │    │          │    │          │    │          │    │      │ │
│  │ Detect & │    │ Process  │    │ Fetch    │    │ AI       │    │Store │ │
│  │ Validate │    │ Media    │    │ Correct  │    │ Evaluate │    │  +   │ │
│  │          │    │          │    │ Answer   │    │          │    │Notify│ │
│  └──────────┘    └──────────┘    └──────────┘    └──────────┘    └──────┘ │
│       │               │               │               │              │     │
│       ▼               ▼               ▼               ▼              ▼     │
│  Check file      If VIDEO:       Query Bedrock    Send to         Save to  │
│  type + size     • Extract       Knowledge Base   Claude AI:      DynamoDB │
│  Is it valid?      audio         "What is the     • Audio         Send     │
│                  • Extract       correct answer    • Frames        push/    │
│  If AUDIO:         frames        for question X?" • Question      email    │
│  • Use as-is    (using FFmpeg)                    • Correct ans            │
│                                  Returns:          • Instructions           │
│                  If AUDIO:       Reference text                             │
│                  • Skip frames                    Returns:                  │
│                  • Audio ready                    Scores +                  │
│                                                   Feedback                  │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Complete Request Flow — Video Submission

```
Trainee          App          API Gateway    Lambda         S3          Step Functions    Bedrock KB     Claude AI      DynamoDB     SNS/SES
  │               │               │            │            │               │               │              │              │            │
  │──Open app────▶│               │            │            │               │               │              │              │            │
  │               │──GET questions─▶───────────▶│            │               │               │              │              │            │
  │               │◀──questions list────────────┤            │               │               │              │              │            │
  │               │               │            │            │               │               │              │              │            │
  │──Record video─▶               │            │            │               │               │              │              │            │
  │               │──GET upload URL▶───────────▶│            │               │               │              │              │            │
  │               │◀──presigned URL────────────┤            │               │               │              │              │            │
  │               │──Upload video directly──────────────────▶│               │               │              │              │            │
  │               │               │            │            │               │               │              │              │            │
  │               │               │            │            │──Event────────▶│               │              │              │            │
  │               │               │            │            │               │               │              │              │            │
  │               │               │            │            │               │──Detect type───▶              │              │            │
  │               │               │            │            │               │  (it's video)  │              │              │            │
  │               │               │            │            │               │               │              │              │            │
  │               │               │            │            │               │──Extract audio + frames──────▶              │            │
  │               │               │            │            │               │               │              │              │            │
  │               │               │            │            │               │──Get correct answer──────────▶│              │            │
  │               │               │            │            │               │◀──Reference answer────────────┤              │            │
  │               │               │            │            │               │               │              │              │            │
  │               │               │            │            │               │──Evaluate (audio+frames+ref)────────────────▶│            │
  │               │               │            │            │               │◀──Scores + Feedback──────────────────────────┤            │
  │               │               │            │            │               │               │              │              │            │
  │               │               │            │            │               │──Store results─────────────────────────────────────────▶│
  │               │               │            │            │               │──Send notification────────────────────────────────────────▶
  │               │               │            │            │               │               │              │              │            │
  │◀───────────────Push: "Your results are ready"───────────────────────────────────────────────────────────────────────────────────────┤
  │               │               │            │            │               │               │              │              │            │
  │──View results─▶──GET /my/results──────────▶│──Query─────────────────────────────────────────────────────────────────▶│            │
  │◀──────────────Scores + Feedback────────────┤            │               │               │              │              │            │
```

---

# PART 4: AWS SERVICES EXPLAINED

---

## All Services — What Each One Does

```
┌─────────────────────────────────────────────────────────────────────────┐
│                                                                          │
│  SERVICE              WHAT IT DOES (in simple terms)                     │
│                                                                          │
│  ─────────────────────────────────────────────────────────────────────  │
│                                                                          │
│  CloudFront           Hosts your web apps (admin + trainee websites)    │
│                       Makes them load fast from anywhere in the world   │
│                                                                          │
│  Cognito              Login system. Handles signup, login, passwords,   │
│                       forgot password, OTP verification                  │
│                                                                          │
│  API Gateway          The "front door" of your backend.                 │
│                       Apps send requests here, it routes to Lambda      │
│                                                                          │
│  Lambda               Small pieces of code that run on-demand.          │
│                       No server to manage. You pay only when it runs.   │
│                       Each "function" does one specific job.            │
│                                                                          │
│  S3                   File storage. Stores video/audio recordings,      │
│                       knowledge base documents, extracted frames        │
│                                                                          │
│  DynamoDB             Database. Stores questions, assignments,          │
│                       results, user profiles. Fast and scalable.        │
│                                                                          │
│  Step Functions       Pipeline manager. Controls the sequence:          │
│                       "first validate, then process, then evaluate..."  │
│                       If one step fails, it handles retry/error.        │
│                                                                          │
│  EventBridge          Event router. When file uploads to S3, it         │
│                       triggers the Step Functions pipeline.             │
│                                                                          │
│  Bedrock (KB)         Knowledge Base. Stores your correct answers.      │
│                       AI searches here to find the reference answer.    │
│                                                                          │
│  Bedrock (Claude)     The AI brain. Claude model that can understand    │
│                       audio + images. Evaluates trainee responses.      │
│                                                                          │
│  SNS                  Push notifications + SMS to trainee               │
│                                                                          │
│  SES                  Email notifications to admin                      │
│                                                                          │
│  WAF                  Security wall. Protects API from attacks          │
│                                                                          │
│  KMS                  Encryption. Makes sure all data is encrypted     │
│                                                                          │
│  IAM                  Permissions. Controls who can access what         │
│                                                                          │
│  CloudWatch           Monitoring. Logs, error alerts, dashboards        │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## AWS Services Visual Map (Grouped)

```
┌─────────────────────────────────────────────────────────────────────────┐
│                                                                          │
│  ╔══════════════════════════════════════════════════════╗                │
│  ║  USER-FACING (what users interact with)              ║                │
│  ║                                                      ║                │
│  ║  CloudFront ─── Web app hosting                     ║                │
│  ║  Cognito ────── Login/signup                        ║                │
│  ║  API Gateway ── API endpoints                       ║                │
│  ╚══════════════════════════════════════════════════════╝                │
│                                                                          │
│  ╔══════════════════════════════════════════════════════╗                │
│  ║  BACKEND LOGIC (runs automatically)                  ║                │
│  ║                                                      ║                │
│  ║  Lambda ─────── Business logic code                 ║                │
│  ║  Step Functions  Pipeline orchestration             ║                │
│  ║  EventBridge ── Triggers pipeline on upload         ║                │
│  ╚══════════════════════════════════════════════════════╝                │
│                                                                          │
│  ╔══════════════════════════════════════════════════════╗                │
│  ║  DATA STORAGE                                        ║                │
│  ║                                                      ║                │
│  ║  S3 ──────────── Files (video, audio, docs)         ║                │
│  ║  DynamoDB ────── Structured data (questions, scores)║                │
│  ╚══════════════════════════════════════════════════════╝                │
│                                                                          │
│  ╔══════════════════════════════════════════════════════╗                │
│  ║  AI / EVALUATION                                     ║                │
│  ║                                                      ║                │
│  ║  Bedrock KB ──── Your correct answers (RAG)         ║                │
│  ║  Bedrock Claude ─ Evaluates responses (multimodal)  ║                │
│  ╚══════════════════════════════════════════════════════╝                │
│                                                                          │
│  ╔══════════════════════════════════════════════════════╗                │
│  ║  NOTIFICATIONS                                       ║                │
│  ║                                                      ║                │
│  ║  SNS ──────── Push notifications + SMS              ║                │
│  ║  SES ──────── Email                                 ║                │
│  ╚══════════════════════════════════════════════════════╝                │
│                                                                          │
│  ╔══════════════════════════════════════════════════════╗                │
│  ║  SECURITY + MONITORING                               ║                │
│  ║                                                      ║                │
│  ║  WAF ──────── API protection                        ║                │
│  ║  KMS ──────── Data encryption                       ║                │
│  ║  IAM ──────── Access permissions                    ║                │
│  ║  CloudWatch ── Logs + alerts                        ║                │
│  ╚══════════════════════════════════════════════════════╝                │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

# PART 5: IMPLEMENTATION CODE & DETAILS

---

## API Design

### Admin APIs

| Method | Endpoint | What It Does |
|--------|----------|-------------|
| POST | /questions | Create a new clinical scenario question |
| GET | /questions | List all questions |
| PUT | /questions/{id} | Edit a question |
| DELETE | /questions/{id} | Delete a question |
| POST | /assignments | Assign questions to trainee(s) |
| GET | /results | Get all trainee results (filterable) |
| GET | /results/{traineeId} | Get specific trainee's results |
| POST | /knowledge-base/upload | Upload correct answer documents |
| GET | /dashboard/stats | Get aggregated statistics |

### Trainee APIs

| Method | Endpoint | What It Does |
|--------|----------|-------------|
| GET | /my/assignments | See assigned question sets |
| GET | /my/questions/{assignmentId} | Get questions for an assignment |
| POST | /my/responses/upload-url | Get secure upload URL for recording |
| POST | /my/responses | Confirm upload complete, start processing |
| GET | /my/results | View own results and feedback |

---

## Database Tables (DynamoDB)

### Questions Table

```
PK: QUESTION#<questionId>
SK: META

Fields:
- questionId (UUID)
- adminId (who created it)
- questionText ("A 35-year-old patient comes in with...")
- category ("periapical", "orthodontics", "hygiene", etc.)
- difficulty ("easy" | "medium" | "hard")
- expectedKeyPoints (["Diagnose abscess", "Prescribe antibiotics", ...])
- totalPoints (number)
- passingScore (number)
- createdAt (timestamp)
```

### Assignments Table

```
PK: ASSIGNMENT#<assignmentId>
SK: TRAINEE#<traineeId>

Fields:
- assignmentId (UUID)
- adminId (who assigned)
- traineeId (who receives)
- questionIds (list of question IDs)
- status ("sent" | "in-progress" | "completed")
- dueDate (timestamp)
- sentAt (timestamp)
```

### Responses Table

```
PK: RESPONSE#<responseId>
SK: QUESTION#<questionId>

Fields:
- responseId (UUID)
- traineeId
- assignmentId
- questionId
- s3Key (path to video/audio file in S3)
- fileType ("video/mp4" | "audio/mp3" | etc.)
- duration (seconds)
- status ("uploaded" | "processing" | "evaluating" | "completed" | "failed")
- uploadedAt (timestamp)
```

### Results Table

```
PK: RESULT#<resultId>
SK: TRAINEE#<traineeId>

Fields:
- resultId (UUID)
- responseId
- traineeId
- questionId
- inputType ("video" | "audio")
- correctness (object: score, total, details)
- confidence (object: score, total, details)
- communication (object: score, total, details)
- bodyLanguage (object: score, total, details — null if audio-only)
- completeness (object: score, total, details)
- overallPercentage (number)
- feedback (string — AI-generated)
- evaluatedAt (timestamp)
```

---

## Lambda Functions

| # | Function | Trigger | Job |
|---|----------|---------|-----|
| 1 | question-manager | API Gateway | Create/read/update/delete questions |
| 2 | assignment-manager | API Gateway | Assign questions to trainees |
| 3 | upload-url-generator | API Gateway | Generate presigned S3 URL for upload |
| 4 | response-handler | API Gateway | Record upload metadata |
| 5 | detect-file-type | Step Functions | Check if file is video or audio |
| 6 | media-processor | Step Functions | Extract audio + frames (FFmpeg) |
| 7 | fetch-reference | Step Functions | Query Bedrock KB for correct answer |
| 8 | evaluate-response | Step Functions | Send to Claude AI for evaluation |
| 9 | store-results | Step Functions | Save scores to DynamoDB |
| 10 | send-notifications | Step Functions | Push/email notifications |
| 11 | results-fetcher | API Gateway | Query results for dashboard |
| 12 | kb-sync | API Gateway / S3 | Trigger KB re-index when docs uploaded |

---

## AI Evaluation Code (Python)

### detect-file-type Lambda

```python
def lambda_handler(event, context):
    """Detect if uploaded file is video or audio."""
    
    file_type = event['fileType']  # e.g. "video/mp4" or "audio/mp3"
    
    video_types = ['video/mp4', 'video/webm', 'video/quicktime']
    audio_types = ['audio/mpeg', 'audio/wav', 'audio/mp4', 'audio/webm', 'audio/m4a']
    
    if file_type in video_types:
        mode = "video"
    elif file_type in audio_types:
        mode = "audio"
    else:
        raise Exception(f"Unsupported file type: {file_type}")
    
    return {
        **event,
        'processingMode': mode  # "video" or "audio"
    }
```

### media-processor Lambda (FFmpeg)

```python
import subprocess
import boto3

s3 = boto3.client('s3')

def lambda_handler(event, context):
    """Extract audio and video frames from uploaded file."""
    
    mode = event['processingMode']
    s3_key = event['s3Key']
    response_id = event['responseId']
    
    # Download file from S3
    local_file = f'/tmp/{response_id}'
    s3.download_file('responses-bucket', s3_key, local_file)
    
    # Extract audio (for both video and audio files)
    audio_path = f'/tmp/{response_id}.mp3'
    subprocess.run([
        'ffmpeg', '-i', local_file,
        '-vn', '-acodec', 'libmp3lame',
        audio_path
    ])
    
    # Upload extracted audio to S3
    audio_s3_key = f'processed/{response_id}/audio.mp3'
    s3.upload_file(audio_path, 'responses-bucket', audio_s3_key)
    
    frames_s3_keys = []
    
    if mode == "video":
        # Extract frames every 15 seconds
        subprocess.run([
            'ffmpeg', '-i', local_file,
            '-vf', 'fps=1/15',  # 1 frame every 15 seconds
            f'/tmp/{response_id}_frame_%03d.jpg'
        ])
        
        # Upload frames to S3
        import glob
        for frame_file in sorted(glob.glob(f'/tmp/{response_id}_frame_*.jpg')):
            frame_name = frame_file.split('/')[-1]
            frame_key = f'processed/{response_id}/frames/{frame_name}'
            s3.upload_file(frame_file, 'responses-bucket', frame_key)
            frames_s3_keys.append(frame_key)
    
    return {
        **event,
        'audioS3Key': audio_s3_key,
        'framesS3Keys': frames_s3_keys  # empty list if audio-only
    }
```

### fetch-reference Lambda

```python
import boto3

bedrock_agent = boto3.client('bedrock-agent-runtime')

def lambda_handler(event, context):
    """Get the correct answer from Knowledge Base."""
    
    question_text = event['questionText']
    
    response = bedrock_agent.retrieve(
        knowledgeBaseId='YOUR_KB_ID',
        retrievalQuery={'text': question_text},
        retrievalConfiguration={
            'vectorSearchConfiguration': {'numberOfResults': 3}
        }
    )
    
    # Combine top results
    reference_texts = []
    for result in response['retrievalResults']:
        reference_texts.append(result['content']['text'])
    
    return {
        **event,
        'referenceAnswer': "\n".join(reference_texts)
    }
```

### evaluate-response Lambda (THE CORE — Sends to Claude)

```python
import boto3
import json
import base64

bedrock = boto3.client('bedrock-runtime')
s3 = boto3.client('s3')

def lambda_handler(event, context):
    """
    THE MAIN EVALUATION STEP.
    Sends audio + video frames + reference answer to Claude.
    Claude evaluates and returns scores.
    """
    
    mode = event['processingMode']  # "video" or "audio"
    question_text = event['questionText']
    reference_answer = event['referenceAnswer']
    audio_s3_key = event['audioS3Key']
    frames_s3_keys = event.get('framesS3Keys', [])
    
    # Download and encode audio
    audio_obj = s3.get_object(Bucket='responses-bucket', Key=audio_s3_key)
    audio_base64 = base64.b64encode(audio_obj['Body'].read()).decode()
    
    # Build message content
    content = []
    
    # Add evaluation instructions
    if mode == "video":
        instructions = f"""You are evaluating a TRAINEE DENTIST's video response.

CLINICAL SCENARIO QUESTION:
{question_text}

CORRECT/EXPECTED ANSWER (from our knowledge base):
{reference_answer}

EVALUATION INSTRUCTIONS:
Listen to the audio and look at the video frames. Evaluate on these dimensions:

1. CORRECTNESS (0-10): Are the clinical answers accurate compared to the reference?
2. CONFIDENCE (0-10): How confidently do they speak? Check for hesitation, filler words (um, uh), pauses, shaky voice.
3. COMMUNICATION (0-10): Is the explanation clear, well-structured, and easy to understand? Even if the answer is correct, poor communication gets a lower score.
4. BODY LANGUAGE (0-10): From the video frames - assess eye contact, posture, hand gestures, professional demeanor.
5. COMPLETENESS (0-10): Did they cover all key points from the reference answer?

IMPORTANT: 
- If communication is poor but answer is correct, still give high correctness but low communication.
- Confidence is about HOW they deliver, not WHAT they say.
- Be encouraging in feedback — these are learners.

Respond ONLY in this JSON format:
{{
  "correctness": {{"score": 0, "total": 10, "details": "explanation"}},
  "confidence": {{"score": 0, "total": 10, "details": "explanation"}},
  "communication": {{"score": 0, "total": 10, "details": "explanation"}},
  "body_language": {{"score": 0, "total": 10, "details": "explanation"}},
  "completeness": {{"score": 0, "total": 10, "details": "explanation"}},
  "overall_percentage": 0,
  "feedback": "2-3 sentence encouraging feedback with specific improvement tips"
}}"""
    else:
        instructions = f"""You are evaluating a TRAINEE DENTIST's audio response.

CLINICAL SCENARIO QUESTION:
{question_text}

CORRECT/EXPECTED ANSWER (from our knowledge base):
{reference_answer}

EVALUATION INSTRUCTIONS:
Listen to the audio recording. Evaluate on these dimensions:

1. CORRECTNESS (0-10): Are the clinical answers accurate compared to the reference?
2. CONFIDENCE (0-10): How confidently do they speak? Check for hesitation, filler words (um, uh), pauses, shaky voice.
3. COMMUNICATION (0-10): Is the explanation clear, well-structured, and easy to understand?
4. COMPLETENESS (0-10): Did they cover all key points from the reference answer?

NOTE: Body language cannot be assessed (audio-only submission).

IMPORTANT:
- If communication is poor but answer is correct, still give high correctness but low communication.
- Confidence is about HOW they deliver, not WHAT they say.
- Be encouraging in feedback — these are learners.

Respond ONLY in this JSON format:
{{
  "correctness": {{"score": 0, "total": 10, "details": "explanation"}},
  "confidence": {{"score": 0, "total": 10, "details": "explanation"}},
  "communication": {{"score": 0, "total": 10, "details": "explanation"}},
  "body_language": null,
  "completeness": {{"score": 0, "total": 10, "details": "explanation"}},
  "overall_percentage": 0,
  "feedback": "2-3 sentence encouraging feedback with specific improvement tips"
}}"""
    
    content.append({"type": "text", "text": instructions})
    
    # Add audio
    content.append({
        "type": "audio",
        "source": {
            "type": "base64",
            "media_type": "audio/mp3",
            "data": audio_base64
        }
    })
    
    # Add video frames (only if video mode)
    if mode == "video":
        for frame_key in frames_s3_keys[:10]:  # Max 10 frames
            frame_obj = s3.get_object(Bucket='responses-bucket', Key=frame_key)
            frame_base64 = base64.b64encode(frame_obj['Body'].read()).decode()
            content.append({
                "type": "image",
                "source": {
                    "type": "base64",
                    "media_type": "image/jpeg",
                    "data": frame_base64
                }
            })
    
    # Call Claude
    response = bedrock.invoke_model(
        modelId='anthropic.claude-3-5-sonnet-20241022-v2:0',
        contentType='application/json',
        accept='application/json',
        body=json.dumps({
            "anthropic_version": "bedrock-2023-05-31",
            "max_tokens": 2048,
            "messages": [{"role": "user", "content": content}]
        })
    )
    
    result = json.loads(response['body'].read())
    evaluation = json.loads(result['content'][0]['text'])
    
    return {
        'responseId': event['responseId'],
        'traineeId': event['traineeId'],
        'questionId': event['questionId'],
        'assignmentId': event['assignmentId'],
        'inputType': mode,
        'evaluation': evaluation
    }
```

---

## Knowledge Base — What to Upload

### Example Reference Answer Document

```json
{
  "question_id": "Q001",
  "question": "A 35-year-old patient comes with severe throbbing pain in lower right molar, facial swelling, and mild fever. What is your diagnosis and treatment plan?",
  "category": "endodontics",
  "correct_answer": "Diagnosis: Periapical abscess. Immediate management: prescribe antibiotics (Amoxicillin 500mg TDS for 5 days), analgesics (Ibuprofen 400mg). If swelling is fluctuant, perform incision and drainage. Definitive treatment: root canal therapy or extraction depending on tooth restorability. Explain to patient in simple terms with empathy. Schedule follow-up in 48-72 hours.",
  "key_points": [
    "Correctly diagnose periapical abscess",
    "Prescribe appropriate antibiotics",
    "Prescribe pain relief",
    "Mention incision and drainage if fluctuant",
    "Discuss root canal or extraction",
    "Patient communication with empathy",
    "Schedule follow-up"
  ],
  "total_points": 7,
  "passing_score": 5
}
```

### Example Scoring Rubric Document

```json
{
  "category": "endodontics",
  "evaluation_criteria": {
    "excellent": "8-10: Comprehensive, confident, well-structured answer",
    "good": "6-7: Correct answer with minor gaps",
    "needs_work": "4-5: Basic understanding but significant gaps",
    "concerning": "0-3: Major knowledge gaps, needs additional training"
  },
  "confidence_indicators": {
    "high_confidence": "Clear speech, no filler words, direct statements",
    "medium_confidence": "Some hesitation, occasional 'um', but recovers",
    "low_confidence": "Frequent pauses, many filler words, questioning tone"
  }
}
```

---

# PART 6: IMPLEMENTATION PLAN, COST & SECURITY

---

## Implementation Timeline

### Phase 1: Foundation (Week 1-2)

| # | Task | Details |
|---|------|---------|
| 1.1 | AWS Account Setup | Create account, set up billing alerts |
| 1.2 | IAM Roles | Create roles for Lambda, Step Functions, etc. |
| 1.3 | Infrastructure Code | Set up AWS CDK (TypeScript) project |
| 1.4 | Cognito | Create user pools for admin + trainee |
| 1.5 | API Gateway | Deploy API with stub Lambda functions |
| 1.6 | DynamoDB | Create all tables with indexes |
| 1.7 | S3 Buckets | Create buckets with encryption |
| 1.8 | CI/CD | Set up GitHub Actions deployment |

### Phase 2: Admin Features (Week 2-3)

| # | Task | Details |
|---|------|---------|
| 2.1 | Question CRUD | Lambda functions for managing questions |
| 2.2 | Assignment Management | Assign questions to trainees |
| 2.3 | KB Upload | Upload correct answers to S3 + trigger sync |
| 2.4 | Bedrock KB Setup | Create Knowledge Base, connect S3 data source |
| 2.5 | Dashboard API | Results query + aggregation endpoints |
| 2.6 | Admin Web App | React/Next.js admin interface |

### Phase 3: Trainee Features (Week 3-4)

| # | Task | Details |
|---|------|---------|
| 3.1 | Upload URL Generation | Presigned URL for secure file upload |
| 3.2 | Response Handler | Record upload metadata, update status |
| 3.3 | Trainee App | Mobile/web app with video + audio recording |
| 3.4 | Event Trigger | S3 event → EventBridge → Step Functions |
| 3.5 | Results View | Trainee sees scores + feedback |

### Phase 4: AI Evaluation Pipeline (Week 4-5)

| # | Task | Details |
|---|------|---------|
| 4.1 | File Type Detection | Lambda to detect video vs audio |
| 4.2 | Media Processor | Lambda with FFmpeg (extract audio + frames) |
| 4.3 | Reference Fetcher | Lambda to query Bedrock KB |
| 4.4 | Evaluator | Lambda to send to Claude + parse response |
| 4.5 | Results Storage | Lambda to save to DynamoDB |
| 4.6 | Notifications | Lambda to send SNS/SES alerts |
| 4.7 | Step Functions | Deploy complete state machine |
| 4.8 | End-to-End Test | Test full flow: upload → evaluate → results |

### Phase 5: Polish (Week 5-6)

| # | Task | Details |
|---|------|---------|
| 5.1 | Security | WAF rules, encryption, access control |
| 5.2 | Monitoring | CloudWatch dashboards + alarms |
| 5.3 | Error Handling | Retry logic, dead letter queues |
| 5.4 | Performance | Test with concurrent uploads |
| 5.5 | UAT | User acceptance testing with real trainees |

---

## Cost Estimate

**Assumptions: 500 trainees/month, 5 questions each = 2,500 evaluations**

| Service | What It Does | Monthly Cost |
|---------|-------------|-------------|
| Lambda | Backend code execution (~40K invocations) | ~$3 |
| API Gateway | API requests (~80K) | ~$0.30 |
| S3 | Video/audio storage (~100GB) | ~$5 |
| DynamoDB | Database reads/writes (~150K) | ~$3 |
| Bedrock Claude Sonnet | AI evaluation (2,500 calls with audio+images) | ~$50-75 |
| Bedrock Knowledge Bases | Reference answer retrieval (2,500 queries) | ~$25 |
| Step Functions | Pipeline orchestration (2,500 × 6 states) | ~$0.40 |
| Cognito | User authentication (550 users) | Free |
| SNS + SES | Notifications (5,000) | ~$1 |
| CloudFront | Web hosting | ~$3 |
| CloudWatch | Logs + monitoring | ~$5 |
| WAF | API security | ~$6 |
| KMS | Encryption keys | ~$2 |
| **TOTAL** | | **~$105-125/month** |

**Note:** Bedrock (Claude) is the biggest cost. If you need to reduce:
- Use Claude Haiku (cheaper, slightly less capable) instead of Sonnet
- Haiku would reduce AI cost to ~$15-25/month
- Total would drop to ~$65-75/month

---

## Security Design

### Who Can Access What

```
ADMIN (Senior Dentist):
  ✅ Create/edit/delete questions
  ✅ Upload knowledge base documents
  ✅ Assign questions to any trainee
  ✅ View ALL trainees' results
  ✅ View dashboard/analytics
  ❌ Cannot modify trainee accounts

TRAINEE:
  ✅ View OWN assigned questions only
  ✅ Upload recordings for OWN assignments only
  ✅ View OWN results only
  ❌ Cannot see other trainees' results
  ❌ Cannot access questions not assigned to them
  ❌ Cannot modify questions or KB
```

### Data Protection

| What | How |
|------|-----|
| Videos/audio at rest | Encrypted with AWS KMS |
| Data in transit | TLS 1.2+ (HTTPS everywhere) |
| Database | DynamoDB encryption enabled |
| API access | JWT tokens from Cognito |
| File uploads | Presigned URLs (expire in 15 min) |
| API protection | WAF (rate limiting, attack prevention) |

---

## Error Handling

| What Goes Wrong | What System Does |
|----------------|-----------------|
| Upload fails | App retries automatically (3 attempts) |
| Invalid file type | Immediate error → trainee sees "Please upload MP4, WebM, MP3, or WAV" |
| File too large (>500MB) | Rejected at upload URL generation step |
| FFmpeg extraction fails | Retry once → if still fails, mark as "processing-error" |
| Knowledge Base has no match | Use general evaluation without reference, flag for admin |
| Claude AI fails/timeout | Retry 2 times → if fails, mark as "needs-manual-review" |
| Notification fails | Goes to dead letter queue, retried later |

---

## Monitoring

| What to Watch | Alert When |
|---------------|-----------|
| Lambda errors | Error rate > 5% |
| Pipeline failures | Any Step Functions failure |
| API errors | 5xx response rate > 1% |
| Upload failures | More than 10 failures in 5 minutes |
| AI response time | Evaluation takes > 60 seconds |
| Cost | Daily spend exceeds threshold |

---

## Step Functions State Machine Definition

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
      "Next": "FetchReference",
      "Catch": [{"ErrorEquals": ["States.ALL"], "Next": "MarkFailed"}]
    },
    "FetchReference": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:...:fetch-reference",
      "Next": "EvaluateResponse",
      "Catch": [{"ErrorEquals": ["States.ALL"], "Next": "MarkFailed"}]
    },
    "EvaluateResponse": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:...:evaluate-response",
      "Next": "StoreResults",
      "Retry": [{"ErrorEquals": ["States.ALL"], "MaxAttempts": 2, "BackoffRate": 2}],
      "Catch": [{"ErrorEquals": ["States.ALL"], "Next": "MarkFailed"}]
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
      "Type": "Task",
      "Resource": "arn:aws:lambda:...:queue-notification-retry",
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

## Technology Stack

| Layer | Technology | Why |
|-------|-----------|-----|
| Frontend (Admin) | React / Next.js | Modern, fast, easy to build dashboards |
| Frontend (Trainee) | React Native or React Web | Cross-platform mobile + web |
| Backend | Python 3.12 (Lambda) | Best for AI/ML integration, Bedrock SDK |
| Infrastructure | AWS CDK (TypeScript) | Repeatable deployments |
| Database | DynamoDB | Serverless, auto-scales, no maintenance |
| File Storage | S3 | Unlimited, cheap, secure |
| AI Evaluation | Amazon Bedrock (Claude Sonnet) | Multimodal — understands audio + images |
| Knowledge Base | Bedrock Knowledge Bases | Managed RAG, no vector DB to maintain |
| Pipeline | AWS Step Functions | Visual workflow, built-in retry/error handling |
| Auth | Amazon Cognito | Managed login, free tier generous |
| Notifications | SNS + SES | Push, SMS, Email |
| CI/CD | GitHub Actions | Free for most use, well-integrated |
| Monitoring | CloudWatch + X-Ray | Native AWS monitoring |

---

## Future Enhancements

| Feature | Description |
|---------|-------------|
| Real-time evaluation | Stream audio live, get instant feedback |
| Peer comparison | "You scored higher than 70% of trainees" |
| Improvement tracking | Graph showing trainee progress over weeks |
| AI-generated follow-ups | "Based on your weak areas, try these questions next" |
| Multi-language | Support trainees in different languages |
| Custom rubrics | Each admin can define their own scoring criteria |
| Batch assignments | Assign same questions to entire cohort at once |
| PDF certificates | Generate certificates for trainees who pass |
| Video playback + annotations | Admin can watch video with AI annotations overlaid |
| Integration with LMS | Connect to existing Learning Management Systems |

---

## How to Test Manually on AWS Console

### Quick Test (5 minutes):

```
1. AWS Console → Amazon Bedrock → Model access → Enable Claude 3.5 Sonnet
2. AWS Console → Amazon Bedrock → Playgrounds → Chat playground
3. Select model: Claude 3.5 Sonnet
4. Upload an audio file (any recording of someone answering a dental question)
5. Type the evaluation prompt (from the code above)
6. Click Send → See AI evaluation result
```

### Full Pipeline Test:

```
1. Upload a video/audio to S3 bucket
2. Check Step Functions → see it trigger automatically
3. Watch each step execute (green = success)
4. Check DynamoDB → see results stored
5. Check email → see notification delivered
```
