# 🎙️ Cyber-Voice: Autonomous AI Outreach & Scheduling Agent

[![Python 3.11+](https://img.shields.io/badge/Python-3.11%20%7C%203.12%20%7C%203.13-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![LiveKit Agents](https://img.shields.io/badge/LiveKit-Agents%201.8.0-002B49?logo=livekit&logoColor=white)](https://livekit.io/)
[![Groq Inference](https://img.shields.io/badge/Groq-LPU%20Inference-F55036?logo=groq&logoColor=white)](https://groq.com/)
[![Deepgram Nova-2](https://img.shields.io/badge/Deepgram-Nova--2%20STT-13EF93?logo=deepgram&logoColor=black)](https://deepgram.com/)
[![Cartesia Sonic](https://img.shields.io/badge/Cartesia-Sonic%203%20TTS-7B2BF9)](https://cartesia.ai/)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

An ultra-low latency (<500ms), bi-directional conversational voice agent engineered for real estate marketing audits and appointment setting. Built with **LiveKit Agents**, powered by **Groq LPU LLM inference**, **Deepgram Nova-2 STT**, and modular TTS backends (**Cartesia Sonic 3**, **ElevenLabs**, and **Deepgram Aura**).

---

## 📑 Table of Contents
- [Architecture & Data Pipeline](#-architecture--data-pipeline)
- [Platform Evaluation & Journey](#-platform-evaluation--journey)
- [Prerequisites & Windows Environment Fixes](#-prerequisites--windows-environment-fixes)
- [Dependencies & Installation](#-dependencies--installation)
- [Environment Variables (.env)](#-environment-variables-env)
- [Code Implementations](#-code-implementations)
  - [Variant A: Cartesia Sonic 3 (Production Standard)](#variant-a-cartesia-sonic-3-production-standard)
  - [Variant B: ElevenLabs Flash v2.5](#variant-b-elevenlabs-flash-v25)
  - [Variant C: Deepgram Aura](#variant-c-deepgram-aura)
- [Diagnostic & Troubleshooting Suite](#-diagnostic--troubleshooting-suite)
- [Future Production Roadmap](#-future-production-roadmap)

---

## 🏛 Architecture & Data Pipeline

```text
[ Human User (Microphone / SIP Trunk) ]
                 │
                 ▼ (WebRTC Audio Stream)
       [ LiveKit Cloud / Server ]
                 │
                 ▼
     [ LiveKit Voice Agent Worker ]
     ├── 1. VAD: Silero VAD (Noise filtering & speech edge detection)
     ├── 2. STT: Deepgram Nova-2 (Fast streaming speech-to-text)
     ├── 3. LLM: Groq LPU (`openai/gpt-oss-120b` @ 300+ tokens/sec)
     ├── 4. TTS: Cartesia Sonic 3 / ElevenLabs / Deepgram Aura
     └── 5. Tools / Webhooks:
              ├── `schedule_zoom_audit` -> Google Sheets / CRM Webhook
              └── `end_call` -> Graceful session teardown
```

---

## 🔍 Platform Evaluation & Journey

During architectural exploration, proprietary and open-source stacks were evaluated:

| Platform | Evaluation Result | Decision |
| :--- | :--- | :--- |
| **Telnyx AI Suite** | Blocked by SIP connection configuration and required provisioned PSTN numbers. High latency (~1800ms). | Replaced by WebRTC LiveKit stack. |
| **Voximplant** | Proprietary CPaaS with mandatory $10.00/month recurring SIP registration fees. | Rejected in favor of open-source stack. |
| **LiveKit Agents** | Open-source, WebRTC-native, sub-500ms pipeline, free developer tier, flexible tool calling. | **Adopted as Core Orchestrator.** |
| **Groq LPU** | Instant token streaming (low latency time-to-first-token) via OpenAI-compatible endpoints. | **Adopted as LLM Brain.** |
| **Cartesia Sonic 3** | Sub-90ms TTS latency, human-grade conversational inflections, stable WebSockets. | **Adopted as Primary Voice Engine.** |

---

## ⚙️ Prerequisites & Windows Environment Fixes

### 1. Windows Execution Alias Hijack Fix
If running `python` opens the Microsoft Store or `pip` is not recognized:
1. Open Windows **Settings** (`Win + I`) $\rightarrow$ **Apps** $\rightarrow$ **Advanced app settings** $\rightarrow$ **App execution aliases**.
2. Toggle **OFF** both `python.exe` and `python3.exe`.
3. Add Python and Scripts to Windows **User PATH**:
   ```text
   C:\Users\<User>\AppData\Local\Programs\Python\Python313\
   C:\Users\<User>\AppData\Local\Programs\Python\Python313\Scripts\
   ```

---

## 📦 Dependencies & Installation

Install all required core packages and plugin providers in a single command:

```bash
pip install livekit-agents \
            livekit-plugins-deepgram \
            livekit-plugins-groq \
            livekit-plugins-silero \
            livekit-plugins-cartesia \
            livekit-plugins-elevenlabs \
            livekit-plugins-openai \
            python-dotenv \
            requests
```

Download Silero VAD weights:
```bash
python -m livekit.agents download-files
```

---

## 🔐 Environment Variables (`.env`)

Create a `.env` file in the root project directory:

```env
# LiveKit Cloud Credentials
LIVEKIT_URL=wss://your-project-id.livekit.cloud
LIVEKIT_API_KEY=your_livekit_api_key
LIVEKIT_API_SECRET=your_livekit_api_secret

# AI Engine API Keys
DEEPGRAM_API_KEY=your_deepgram_api_key
GROQ_API_KEY=gsk_your_groq_api_key
CARTESIA_API_KEY=sk_car_your_cartesia_key
ELEVEN_API_KEY=sk_your_elevenlabs_key

# Webhook Integrations
GOOGLE_SHEET_WEBHOOK=https://script.google.com/macros/s/your_deployment_id/exec
```

---

## 💻 Code Implementations

### Variant A: Cartesia Sonic 3 (Production Standard)
> **Filename:** `agent_cartesia.py`  
> **Features:** <90ms TTS latency, natural breathing/intonation, non-quitting sales rebuttals, Google Sheets webhook integration, and graceful disconnect delay.

```python
import os
import asyncio
import requests
from dotenv import load_dotenv

from livekit.agents import (
    Agent,
    AgentSession,
    JobContext,
    WorkerOptions,
    cli,
    function_tool,
    RunContext,
)
from livekit.plugins import deepgram, groq, silero, cartesia

load_dotenv()

GOOGLE_SHEET_WEBHOOK = os.getenv("GOOGLE_SHEET_WEBHOOK", "")

PROMPT = """# ROLE AND IDENTITY
You are Sarah, an elite outbound marketing consultant for TG Agency. You sound like a warm, confident, sharp, and natural American professional. You speak in concise conversational turns (1-2 sentences max, under 25 words per turn).

# SALES MINDSET & BEHAVIOR
- You NEVER give up or say goodbye on the first "No", "Not interested", or "I'm busy".
- Acknowledge objections in 3 words and immediately pivot to the discovery question.
- Speak in natural American English text ONLY.
- NEVER use filler sounds like "um", "uh", "ah", "er".
- Ask only ONE question at a time.

# SPOKEN EMAIL FORMATTING RULE
- When the user speaks an email (e.g., "john at gmail dot com"), intelligently parse it into standard format ("john@gmail.com") without asking them to re-spell it unless unintelligible.

# CONVERSATION FLOW
1. GREETING: "Hi! This is Sarah with TG Agency. I'll be super quick, I'm calling about your active listing. Do you have 20 seconds?"
2. DISCOVERY / PIVOT: 
   - IF YES: "Awesome. Are you currently editing your own property Reels and managing your CRM late at night, or do you have a dedicated team handling that?"
   - IF NO / BUSY: "I completely understand, I'll be ultra-quick! Just curious—are you editing your own property Reels and handling admin late at night, or do you have a team for that?"
3. VALUE OFFER: "Got it. We actually want to build you a free 30-second custom video sample for your current listing to show you what our team can do."
4. SCHEDULING TIME: "We walk through the video and our system in an 8-minute Zoom preview. What works better for you—tomorrow at 6:00 PM, or the following morning at 8:30 AM?"
5. COLLECT EMAIL: Once a time is chosen, ask: "Perfect! What is the best email address to send the calendar invite to?" (STOP AND WAIT FOR EMAIL).
6. CONFIRMATION & HANG UP: Once email is received:
   - Trigger `schedule_zoom_audit`
   - Say: "Awesome! I just sent the invite to your email. Thanks so much for your time, and talk soon. Goodbye!"
   - Trigger `end_call`
"""

class OutreachAgent(Agent):
    def __init__(self):
        super().__init__(instructions=PROMPT)

    async def on_enter(self) -> None:
        await self.session.generate_reply(
            instructions="Say: 'Hi! This is Sarah with TG Agency. I will be super quick, I am calling about your active property listing. Do you have 20 seconds?'"
        )

    @function_tool()
    async def schedule_zoom_audit(self, context: RunContext, email: str, preferred_time: str) -> str:
        """Schedules a 10-minute Zoom audit, sends invite, and appends to Google Sheet."""
        print(f"\n[APPOINTMENT BOOKED] -> Email: {email} | Time: {preferred_time}\n")
        if GOOGLE_SHEET_WEBHOOK.startswith("http"):
            try:
                requests.post(GOOGLE_SHEET_WEBHOOK, json={"email": email, "time": preferred_time}, timeout=5)
                print("[GOOGLE SHEETS] Successfully logged row.")
            except Exception as e:
                print(f"[GOOGLE SHEETS ERROR] {e}")
        return f"Successfully booked Zoom audit for {email} at {preferred_time}."

    @function_tool()
    async def end_call(self, context: RunContext) -> str:
        """Terminates session after saying final goodbye."""
        print("\n[CALL ENDED] -> Graceful teardown scheduled.\n")
        asyncio.create_task(self._hangup_after_delay())
        return "Call disconnected."

    async def _hangup_after_delay(self):
        await asyncio.sleep(6.0)
        await self.session.aclose()

async def entrypoint(ctx: JobContext):
    await ctx.connect()
    vad_plugin = silero.VAD.load(min_speech_duration=0.2, min_silence_duration=0.5, activation_threshold=0.65)
    
    session = AgentSession(
        vad=vad_plugin,
        stt=deepgram.STT(model="nova-2"),
        llm=groq.LLM(model="openai/gpt-oss-120b", api_key=os.getenv("GROQ_API_KEY")),
        tts=cartesia.TTS(model="sonic-3", voice="db6b0ed5-d5d3-463d-ae85-518a07d3c2b4", api_key=os.getenv("CARTESIA_API_KEY")),
    )

    agent = OutreachAgent()
    await session.start(room=ctx.room, agent=agent)

if __name__ == "__main__":
    cli.run_app(WorkerOptions(entrypoint_fnc=entrypoint))
```

---

### Variant B: ElevenLabs Flash v2.5
> **Filename:** `agent_elevenlabs.py`  
> **Key Configuration:** Employs `livekit-plugins-elevenlabs` with `model="eleven_flash_v2_5"` and `voice_id="21m00Tcm4TlvDq8ikWAM"` (Rachel).

```python
from livekit.plugins import elevenlabs

# In AgentSession:
tts = elevenlabs.TTS(
    model="eleven_flash_v2_5",
    voice_id="21m00Tcm4TlvDq8ikWAM",  # Rachel - American Female
    api_key=os.getenv("ELEVEN_API_KEY"),
)
```

---

### Variant C: Deepgram Aura
> **Filename:** `agent_deepgram.py`  
> **Key Configuration:** Zero extra API keys needed, lightweight STT+TTS combination.

```python
from livekit.plugins import deepgram

# In AgentSession:
tts = deepgram.TTS(model="aura-asteria-en") # or "aura-perseus-en" for American Male
```

---

## 🛠 Diagnostic & Troubleshooting Suite

These single-line scripts can be used to quickly verify credentials and list active models:

### 1. Test Groq Authentication & Available Models
```bash
python -c "import os, requests; from dotenv import load_dotenv; load_dotenv(); res = requests.get('https://api.groq.com/openai/v1/models', headers={'Authorization': f'Bearer {os.getenv(\"GROQ_API_KEY\")}'}); print('STATUS:', res.status_code); print('MODELS:', [m['id'] for m in res.json().get('data', [])])"
```

### 2. Test ElevenLabs Tier & Character Credits
```bash
python -c "import os, requests; from dotenv import load_dotenv; load_dotenv(); res = requests.get('https://api.elevenlabs.io/v1/user', headers={'xi-api-key': os.getenv('ELEVEN_API_KEY')}); print('STATUS:', res.status_code); print('TIER:', res.json().get('subscription', {}).get('tier'))"
```

### 3. Test Cartesia API Connection
```bash
python -c "import os, requests; from dotenv import load_dotenv; load_dotenv(); res = requests.get('https://api.cartesia.ai/voices', headers={'X-API-Key': os.getenv('CARTESIA_API_KEY'), 'Cartesia-Version': '2024-06-10'}); print('STATUS:', res.status_code)"
```

### 4. Run the Agent in Development Mode
```bash
python agent_cartesia.py dev
```
*Test live at:* [LiveKit Agents Console](https://cloud.livekit.io/) or [Agents Playground](https://agents-playground.livekit.io/).

---

## 🗺 Future Production Roadmap

- [ ] **Inbound/Outbound Telephony:** Integrate LiveKit SIP Dispatch with Twilio/Telnyx SIP trunking for direct cellular dial-in/dial-out.
- [ ] **24/7 Cloud Deployment:** Containerize worker into Docker and deploy to Linux VPS / Railway / AWS EC2.
- [ ] **CRM Bidirectional Sync:** Direct API integration with HubSpot / Follow Up Boss / GoHighLevel.
- [ ] **Dynamic Lead Personalization:** Pass CSV variables (`{{first_name}}`, `{{property_address}}`) directly into LiveKit room metadata upon call dispatch.
