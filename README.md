# Real-Time-Voice-Intelligence-Platform--Bootcamp
Bootcamp Tutorial — Full Source Code  Build a production-grade voice AI system from scratch using the OpenAI Realtime API. 7 progressive labs · Full Python source · Docker + Kubernetes deployment


> **Build a production-grade voice AI system from scratch using the OpenAI Realtime API.**  
> 7 progressive labs · Full Python source · Docker + Kubernetes deployment

---

## Table of Contents

1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [Quick Start](#quick-start)
4. [Project Structure](#project-structure)
5. [Lab Guide](#lab-guide)
   - [Lab 01 — Hello Realtime](#lab-01--hello-realtime)
   - [Lab 02 — Live Microphone Pipeline](#lab-02--live-microphone-pipeline)
   - [Lab 03 — Translation Engine & Model Router](#lab-03--translation-engine--model-router)
   - [Lab 04 — Voice-to-Action (Tool Calling)](#lab-04--voice-to-action-tool-calling)
   - [Lab 05 — Emotional Intelligence Engine](#lab-05--emotional-intelligence-engine)
   - [Lab 06 — Telephony Bridge (Twilio)](#lab-06--telephony-bridge-twilio)
   - [Lab 07 — Docker, Kubernetes & Helm](#lab-07--docker-kubernetes--helm)
6. [Audio Format Reference](#audio-format-reference)
7. [Troubleshooting](#troubleshooting)
8. [Cost Estimates](#cost-estimates)
9. [Licence](#licence)

---

## Overview

This repository contains every line of code from the **Real-Time Voice AI Bootcamp** series. The platform integrates three OpenAI Realtime API capabilities — high-reasoning conversation, real-time multilingual translation, and live transcription — into a single, production-ready system that supports both web clients and PSTN telephone calls.

| Feature | Technology |
|---|---|
| Core AI | OpenAI Realtime API (gpt-4o-realtime-preview) |
| Voice Activity Detection | Energy-based Python VAD |
| Language Detection | `langdetect` |
| Emotion Analysis | HuggingFace `cardiffnlp/twitter-roberta-base-sentiment-latest` |
| Telephony | Twilio Media Streams |
| API Server | FastAPI + Uvicorn |
| Session State | Redis |
| Transcript Store | PostgreSQL |
| Container Orchestration | Kubernetes + Helm |

---

## Prerequisites

| Tool | Version | Notes |
|---|---|---|
| Python | 3.12+ | Use `pyenv` for version management |
| Node.js | 22+ | Only needed for gateway service |
| Docker Desktop | Latest | For Lab 07 |
| kubectl | 1.29+ | For Kubernetes deployment |
| Helm | 3.14+ | For Kubernetes deployment |
| ngrok | Latest | For Lab 06 (Twilio webhook) |
| OpenAI API key | — | Must have Realtime API access enabled |
| Twilio account | — | Free trial sufficient for Lab 06 |

**Audio hardware:** Labs 02–06 require a working microphone and speakers (or headphones).

---

## Quick Start

```bash
# 1. Clone and enter the project
git clone https://github.com/yourorg/voice-ai-bootcamp.git
cd voice-ai-bootcamp

# 2. Create virtual environment
python -m venv .venv
source .venv/bin/activate      # macOS / Linux
# .venv\Scripts\activate       # Windows

# 3. Install all dependencies
pip install -r requirements.txt

# 4. Set your API key
cp .env.example .env
# Edit .env and add your OPENAI_API_KEY

# 5. Run the first lab to verify everything works
cd lab01
python lab01_hello.py
```

Expected output:
```
[*] Connecting to OpenAI Realtime API...
[*] Session configured.
Hello! How can I assist you today?
[✓] Full response: Hello! How can I assist you today?
[✓] Response complete.
```

---

## Project Structure

```
voice-ai-bootcamp/
├── README.md
├── requirements.txt          # All Python dependencies
├── .env.example              # Environment variable template
├── .gitignore
│
├── lab01/                    # Hello Realtime
│   ├── lab01_hello.py        # Basic connection + text streaming
│   └── lab01_audio_save.py   # Capture audio response to WAV
│
├── lab02/                    # Live Audio Pipeline
│   ├── lab02_mic_vad.py      # Voice Activity Detection module
│   └── lab02_full_duplex.py  # Real-time mic in → AI out
│
├── lab03/                    # Translation Engine
│   ├── lab03_lang_detect.py  # Language detection utilities
│   ├── lab03_model_router.py # ModelRouter: CONVERSE/TRANSLATE/TRANSCRIBE
│   └── lab03_translation_demo.py  # Adaptive language switching demo
│
├── lab04/                    # Voice-to-Action
│   ├── lab04_tool_registry.py  # Tool implementations + JSON schemas
│   └── lab04_voice_agent.py    # Voice agent with live tool execution
│
├── lab05/                    # Emotional Intelligence
│   ├── lab05_emotion_engine.py  # Sentiment classifier + tone directives
│   └── lab05_emotion_agent.py   # Emotion-aware voice agent
│
├── lab06/                    # Telephony Bridge
│   └── lab06_twilio_bridge.py  # FastAPI + Twilio Media Streams
│
└── lab07/                    # Production Deployment
    ├── services/
    │   ├── orchestrator/
    │   │   ├── main.py          # FastAPI session orchestrator
    │   │   ├── requirements.txt
    │   │   └── Dockerfile
    │   └── emotion-engine/      # (scaffold — extend from lab05)
    └── infra/
        ├── docker-compose.yml   # Full local stack
        ├── init.sql             # PostgreSQL schema
        └── helm/voice-ai/       # Helm chart for Kubernetes
            ├── Chart.yaml
            ├── values.yaml
            └── templates/
                └── orchestrator-deployment.yaml
```

---

## Lab Guide

### Lab 01 — Hello Realtime

**Goal:** Establish your first connection to the OpenAI Realtime API and receive a streamed response.

```bash
cd lab01
python lab01_hello.py          # text response
python lab01_audio_save.py     # + save audio as output.wav
```

**Key concepts:**
- `client.beta.realtime.connect()` — opens a persistent WebSocket
- `conn.session.update()` — configures modalities, voice, audio format
- `response.text.delta` events — streaming text tokens
- `response.audio.delta` events — base64 PCM16 audio chunks

---

### Lab 02 — Live Microphone Pipeline

**Goal:** Capture live microphone input, apply VAD, and have a real-time spoken conversation.

```bash
cd lab02

# Step 1: verify VAD thresholds with your microphone
python lab02_mic_vad.py

# Step 2: run the full duplex demo
python lab02_full_duplex.py
```

**Tuning VAD thresholds** (in `lab02_mic_vad.py`):

| Constant | Default | Effect |
|---|---|---|
| `SPEECH_THRESHOLD` | 800 | Raise if false triggers in noisy environment |
| `SILENCE_THRESHOLD` | 500 | Lower if AI triggers too slowly |
| `SILENCE_FRAMES` | 40 | Increase (×20ms) for longer pauses before response |

**Audio specs:**
- Input to API: **PCM16 / 16 kHz / Mono**
- Output from API: **PCM16 / 24 kHz / Mono**

---

### Lab 03 — Translation Engine & Model Router

**Goal:** Auto-detect spoken language and switch to translation mode transparently.

```bash
cd lab03
python lab03_lang_detect.py       # self-test language detector
python lab03_model_router.py      # self-test router
python lab03_translation_demo.py  # live adaptive demo
```

**Supported translation output languages:**
`en`, `es`, `fr`, `de`, `ja`, `ko`, `zh-cn`, `pt`, `ar`, `hi`, `it`, `nl`, `pl`

**How the Model Router works:**

```python
from lab03_model_router import ModelRouter, SessionConfig, ModelMode

router = ModelRouter()

# Normal conversation
model, params = router.resolve(SessionConfig(mode=ModelMode.CONVERSE))

# French → English translation
model, params = router.resolve(SessionConfig(
    mode=ModelMode.TRANSLATE,
    source_lang="fr",
    target_lang="en",
))
```

---

### Lab 04 — Voice-to-Action (Tool Calling)

**Goal:** The AI executes real tasks (calendar, CRM) mid-conversation without interrupting the audio stream.

```bash
cd lab04
python lab04_tool_registry.py  # self-test all three tools
python lab04_voice_agent.py    # live voice agent with tools
```

**Available tools:**

| Tool | Example voice command |
|---|---|
| `check_calendar` | "What's on my calendar tomorrow?" |
| `book_appointment` | "Book a meeting called Sprint Review on May 24 at 10am" |
| `query_order_status` | "What's the status of order ORD-1002?" |

**Adding a new tool:**

1. Implement the function in `lab04_tool_registry.py`
2. Add it to `_TOOL_REGISTRY`
3. Add its JSON Schema to `TOOL_SCHEMAS`
4. Restart the agent — no other changes needed

---

### Lab 05 — Emotional Intelligence Engine

**Goal:** The AI detects frustration, satisfaction, urgency etc. and adapts its spoken tone automatically.

```bash
cd lab05
python lab05_emotion_engine.py  # self-test classifier
python lab05_emotion_agent.py   # live emotion-adaptive agent
```

**Emotion → Tone mapping:**

| Detected Emotion | AI Behaviour |
|---|---|
| `FRUSTRATED` | Calm, empathetic; acknowledges concerns first |
| `SATISFIED` | Warm, slightly playful; matches positive energy |
| `URGENT` | Direct; skips pleasantries; short sentences |
| `CONFUSED` | Slower; step-by-step; offers to rephrase |
| `ENTHUSIASTIC` | Lively, expressive; high energy |
| `NEUTRAL` | Professional, clear, concise (default) |

The tone directive is refreshed every `TURN_REFRESH_EVERY = 3` turns. Adjust this constant to update more or less frequently.

---

### Lab 06 — Telephony Bridge (Twilio)

**Goal:** A real PSTN phone number answers with your AI agent.

```bash
# Terminal 1: start the server
cd lab06
python lab06_twilio_bridge.py

# Terminal 2: expose via ngrok
ngrok http 8000

# Twilio dashboard:
#   Phone Numbers → Your Number → Voice webhook
#   Set to: https://<ngrok-subdomain>.ngrok.io/incoming-call
```

**Audio conversion chain:**

```
Caller mic → Twilio mulaw/8kHz → [audioop: ulaw2lin + ratecv] → PCM16/16kHz → OpenAI
OpenAI PCM16/24kHz → [audioop: ratecv + lin2ulaw] → mulaw/8kHz → Twilio → Caller speaker
```

**Required environment variables for Lab 06:**
- `OPENAI_API_KEY` — required
- `TWILIO_ACCOUNT_SID` and `TWILIO_AUTH_TOKEN` — only needed if you send outbound calls

---

### Lab 07 — Docker, Kubernetes & Helm

**Goal:** Containerise the platform and deploy to a Kubernetes cluster.

#### Local Docker Compose (full stack)

```bash
cd lab07/infra
docker-compose up --build

# Services started:
#   http://localhost:8000  — Session Orchestrator
#   http://localhost:8001  — Emotion Engine
#   localhost:5432         — PostgreSQL
#   localhost:6379         — Redis
```

#### Production Kubernetes (Helm)

```bash
# 1. Build and push images
docker build -t ghcr.io/yourorg/orchestrator:latest lab07/services/orchestrator
docker push ghcr.io/yourorg/orchestrator:latest

# 2. Create namespace and API key secret
kubectl create namespace voice-ai
kubectl create secret generic voice-ai-secrets \
  --from-literal=openai-api-key=$OPENAI_API_KEY \
  --namespace voice-ai

# 3. Deploy
helm upgrade --install voice-ai ./lab07/infra/helm/voice-ai \
  --namespace voice-ai \
  --set image.tag=latest \
  --set secrets.openaiApiKey=$OPENAI_API_KEY

# 4. Watch rollout
kubectl rollout status deployment/voice-ai-orchestrator -n voice-ai

# 5. Get load balancer IP
kubectl get svc -n voice-ai
```

#### Scaling

The Helm chart configures a Horizontal Pod Autoscaler. Tune in `values.yaml`:

```yaml
autoscaling:
  enabled:                        true
  minReplicas:                    2
  maxReplicas:                    20
  targetCPUUtilizationPercentage: 60
```

---

## Audio Format Reference

| Stage | Format | Sample Rate | Channels |
|---|---|---|---|
| Microphone capture | PCM16 (raw) | 16 kHz | Mono |
| API input | PCM16 (base64) | 16 kHz | Mono |
| API output | PCM16 (base64) | 24 kHz | Mono |
| Twilio PSTN input | mulaw | 8 kHz | Mono |
| Twilio PSTN output | mulaw | 8 kHz | Mono |
| WAV archive | PCM16 | 24 kHz | Mono |

---

## Troubleshooting

| Symptom | Fix |
|---|---|
| AI responds with silence | Verify `output_audio_format: pcm16`; check audio_chunks are non-empty |
| High latency (>800 ms) | Reduce `CHUNK_MS` to 10 ms; check network latency to `api.openai.com` |
| Tool call never fires | Confirm `tool_choice: "auto"`; verify tool schema has `"type": "function"` |
| VAD triggers on background noise | Raise `SPEECH_THRESHOLD` (default 800); consider hardware noise-cancelling |
| Twilio audio is garbled | Ensure mulaw↔PCM16 conversion at correct rates: 8 kHz↔24 kHz |
| Emotion model is slow | Set `device=0` in `lab05_emotion_engine.py` if you have a GPU |
| WebSocket drops mid-session | Implement exponential backoff reconnect; store session state in Redis |
| `audioop` import error (Python 3.13+) | `pip install audioop-lts` |
| `ModuleNotFoundError: sounddevice` | `sudo apt install libportaudio2` then `pip install sounddevice` |

---

## Cost Estimates

Indicative costs for **10,000 minutes of voice conversation per day**:

| Component | Provider | Est. Monthly (USD) |
|---|---|---|
| GPT-Realtime-2 API | OpenAI | $4,200 – $8,000 |
| Translation API | OpenAI | $800 – $1,500 |
| Whisper Transcription | OpenAI | $600 – $1,200 |
| Kubernetes Cluster | AWS EKS | $1,200 – $2,400 |
| PostgreSQL (RDS) | AWS | $300 – $600 |
| Redis (ElastiCache) | AWS | $150 – $300 |
| S3 Audio Archive | AWS | $80 – $200 |
| Telephony (Twilio) | Twilio | $500 – $2,000 |
| **Total** | | **$7,830 – $16,200/month** |

> 💡 **Cost control tip:** Monitor token usage from day one. Set per-tenant budget limits in your OpenAI organisation settings. Use Redis caching for frequently repeated queries.

---

## Licence

MIT Licence — see [LICENSE](LICENSE) for details.

---

*Prepared by **Erica Jayasundera** | Mindview AI | May 2026*
