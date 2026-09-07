# 🎙️ Cyber-Voice: Autonomous AI Outreach & Scheduling Agent

[![Python 3.11+](https://img.shields.io/badge/Python-3.11%20%7C%203.12%20%7C%203.13-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![LiveKit Agents](https://img.shields.io/badge/LiveKit-Agents%201.8.0-002B49?logo=livekit&logoColor=white)](https://livekit.io/)
[![Groq LPU](https://img.shields.io/badge/Groq-LPU%20Inference-F55036?logo=groq&logoColor=white)](https://groq.com/)
[![Deepgram Nova-2](https://img.shields.io/badge/Deepgram-Nova--2%20STT-13EF93?logo=deepgram&logoColor=black)](https://deepgram.com/)
[![Cartesia Sonic 3](https://img.shields.io/badge/Cartesia-Sonic%203%20TTS-7B2BF9)](https://cartesia.ai/)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

An ultra-low latency (<300ms), bi-directional conversational voice AI agent engineered for automated real estate marketing audits and appointment setting. Built with **LiveKit Agents v1.8.0**, powered by **Groq LPU LLM inference**, **Deepgram Nova-2 STT**, and **Cartesia Sonic 3 TTS**.

---

## 📑 Table of Contents
- [Architecture & Data Pipeline](#architecture-data-pipeline)
- [Performance & Engineering Milestones](#performance-engineering-milestones)
- [OSC Sales Framework & Conversation Flow](#osc-sales-framework)
- [Prerequisites & Windows Environment Fixes](#prerequisites-windows-fixes)
- [Dependencies & Installation](#dependencies-installation)
- [Environment Variables (.env)](#environment-variables)
- [CLI Diagnostic & Verification Suite](#cli-diagnostic-suite)
- [Engineering Journal & Debugging Matrix](#debugging-matrix)
- [Future Production Roadmap](#future-roadmap)

---

<a id="architecture-data-pipeline"></a>
## 🏛 Architecture & Data Pipeline

```text
[ Human User (Microphone / WebRTC / Phone SIP) ]
                         │
                         ▼
             [ LiveKit Cloud Engine ]
                         │
                         ▼
        [ Cyber-Voice Python Agent Worker ]
     ├── 1. VAD: Silero VAD (Noise suppression & anti-echo tuning)
     ├── 2. STT: Deepgram Nova-2 (Fast streaming speech-to-text)
     ├── 3. LLM: Groq LPU (`openai/gpt-oss-20b` @ 400+ tokens/sec)
     ├── 4. TTS: Cartesia Sonic 3 ("Skylar" American Female Voice)
     └── 5. Tools & Webhooks:
              ├── `schedule_zoom_audit` -> Non-blocking aiohttp Google Sheets POST
              └── `end_call` -> Graceful teardown buffer
```

---

<a id="performance-engineering-milestones"></a>
## ⚡ Performance & Engineering Milestones

| Metric / Area | Baseline / Initial State | Production Optimization | Improvement |
| :--- | :--- | :--- | :--- |
| **End-to-End Latency** | 1,800ms - 15,000ms | **180ms - 350ms** | **95% Latency Reduction** |
| **Groq TPM Rate Limits** | 429 Errors (8,000 TPM limit on 120B) | **Switched to `openai/gpt-oss-20b` (30,000+ TPM)** | **0% Rate Limit Errors** |
| **Prompt Token Weight** | ~3,800 tokens / request | **~750 tokens / request** | **80% Token Cost Reduction** |
| **HTTP Webhook Overhead** | Sync `requests.post()` blocked event loop | **Async `aiohttp.ClientSession()`** | **Zero Event-Loop Stutter** |
| **Windows Resampler** | C++ DLL assertion crash (`soxr-sys`) | **`SOXR_MAX_THREADS = "1"`** | **100% Windows Stability** |

---

<a id="osc-sales-framework"></a>
## 🎯 OSC Sales Framework & Conversation Flow

The agent ("Sarah") is built on the **OSC Outbound Sales Handbook** for B2B real estate agency outreach:

1. **Permission-Based Hook:** Asks for 20 seconds regarding the agent's active listing.
2. **Showing Bottleneck Discovery:** Discovers what time the real estate agent finishes client showings in the evening before offering Zoom slots.
3. **Choice of Two Close:** Offers specific times ("Tomorrow at 6:00 PM, or Thursday at 8:30 AM").
4. **Spoken Email Normalization:** Parses spoken emails (e.g., `"john at gmail dot com"`) directly into clean string formats (`"john@gmail.com"`).
5. **Solidification (4-Point Lock & Smart Bypass):** Provides confirmation code `COMPASS10`. If the prospect says they don't have a pen, Sarah intelligently bypasses the read-back and confirms via email.
6. **The 2-Goodbye Defense:** Ignores early casual brush-off goodbyes ("Okay thanks bye") with a soft value pivot, only ending the session when the appointment is locked or on a second explicit refusal.

---

<a id="prerequisites-windows-fixes"></a>
## ⚙️ Prerequisites & Windows Environment Fixes

### 1. Windows Execution Alias Hijack Fix
If running `python` opens the Microsoft Store or `pip` is unmapped:
1. Open Windows **Settings** (`Win + I`) $\rightarrow$ **Apps** $\rightarrow$ **Advanced app settings** $\rightarrow$ **App execution aliases**.
2. Toggle **OFF** both `python.exe` and `python3.exe`.
3. Add Python and Scripts to User PATH:
   ```text
   C:\Users\<User>\AppData\Local\Programs\Python\Python313\
   C:\Users\<User>\AppData\Local\Programs\Python\Python313\Scripts\
   ```

---

<a id="dependencies-installation"></a>
## 📦 Dependencies & Installation

Install all required core packages and plugin providers in a single command:

```bash
py -m pip install livekit-agents \
                  livekit-plugins-deepgram \
                  livekit-plugins-groq \
                  livekit-plugins-silero \
                  livekit-plugins-cartesia \
                  livekit-plugins-openai \
                  python-dotenv \
                  aiohttp \
                  requests
```

Download Silero VAD neural network weights:
```bash
py -m livekit.agents download-files
```

---

<a id="environment-variables"></a>
## 🔐 Environment Variables (`.env`)

Create a `.env` file in the root project directory:

```env
# LiveKit Cloud Credentials
LIVEKIT_URL=wss://your-project-id.livekit.cloud
LIVEKIT_API_KEY=your_livekit_api_key
LIVEKIT_API_SECRET=your_livekit_api_secret

# Core AI Service API Keys
DEEPGRAM_API_KEY=your_deepgram_api_key
GROQ_API_KEY=gsk_your_groq_api_key
CARTESIA_API_KEY=sk_car_your_cartesia_key

# Webhook Integrations
GOOGLE_SHEET_WEBHOOK=https://script.google.com/macros/s/your_deployment_id/exec
```

---

<a id="cli-diagnostic-suite"></a>
## 🛠 CLI Diagnostic & Verification Suite

Use these single-line terminal commands to test API credentials:

### 1. Query Active Groq Models & Rate Limit Allowance
```bash
py -c "import requests; res = requests.get('https://api.groq.com/openai/v1/models', headers={'Authorization': 'Bearer YOUR_GROQ_KEY'}); print('GROQ STATUS:', res.status_code); print('AVAILABLE MODELS:', [m['id'] for m in res.json().get('data', [])])"
```

### 2. Verify Cartesia API Connectivity
```bash
py -c "import os, requests; from dotenv import load_dotenv; load_dotenv(); key = os.getenv('CARTESIA_API_KEY'); res = requests.get('https://api.cartesia.ai/voices', headers={'X-API-Key': key or '', 'Cartesia-Version': '2024-06-10'}); print('STATUS:', res.status_code)"
```

### 3. Run the Agent in Development Mode
```bash
py agent.py dev
```

---

<a id="debugging-matrix"></a>
## 🧠 Engineering Journal & Debugging Matrix

| Issue / Exception Encountered | Root Cause | Architectural Resolution |
| :--- | :--- | :--- |
| `TypeError: unexpected keyword 'preemptive_synthesis'` | Parameter removed from `AgentSession` in LiveKit 1.8.0. | Removed parameter; managed stream flow natively via event loop. |
| `APIStatusError: 429 Rate Limit Exceeded (8000 TPM)` | Large 120B model exceeded free-tier token allocation during multi-turn chats. | Switched LLM to `openai/gpt-oss-20b` (30,000+ TPM allowance) and compressed prompt by 80%. |
| `Assertion failed: LSX_FFT_BR == NULL` | Windows multithreading race condition inside native `livekit_ffi.dll` audio resampler. | Set `os.environ["SOXR_MAX_THREADS"] = "1"` at top of script. |
| `resumed false interrupted speech` Stutter Loop | Speaker audio feeding into microphone (acoustic echo) or VAD sensitivity too low. | Set Silero VAD `activation_threshold=0.70` and `min_speech_duration=0.35s`. |
| LLM Outputting Meta-Notes `(Note: waiting for user)` | Completion model trying to finish entire prompt list in a single turn. | Set `temperature=0.2` and enforced strict 1-turn, max 20-word generation protocol. |
| Sync Event-Loop Freeze | `requests.post()` blocked Python `asyncio` event loop during CRM POSTs. | Replaced with non-blocking `aiohttp.ClientSession()` POST requests. |

---

<a id="future-roadmap"></a>
## 🗺 Future Production Roadmap

- [ ] **Inbound/Outbound SIP Trunking:** Connect LiveKit SIP Dispatch with Twilio/Telnyx SIP trunking for cellular dial-in and automated dialer campaigns.
- [ ] **24/7 Cloud Hosting:** Containerize worker into Docker and deploy to AWS EC2 or Railway.
- [ ] **CRM Sync:** Direct API integration with Follow Up Boss and GoHighLevel.
