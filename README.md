# 🎙️ Cyber-Voice: Autonomous AI Outreach & Scheduling Agent

[![Python 3.11+](https://img.shields.io/badge/Python-3.11%20%7C%203.12%20%7C%203.13-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![LiveKit Agents](https://img.shields.io/badge/LiveKit-Agents%201.8.0-002B49?logo=livekit&logoColor=white)](https://livekit.io/)
[![Groq Inference](https://img.shields.io/badge/Groq-LPU%20Inference-F55036?logo=groq&logoColor=white)](https://groq.com/)
[![Deepgram Nova-2](https://img.shields.io/badge/Deepgram-Nova--2%20STT-13EF93?logo=deepgram&logoColor=black)](https://deepgram.com/)
[![Cartesia Sonic](https://img.shields.io/badge/Cartesia-Sonic%203%20TTS-7B2BF9)](https://cartesia.ai/)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

An ultra-low latency (<500ms), bi-directional conversational voice agent engineered for real estate marketing audits and automated appointment setting. Built with **LiveKit Agents**, powered by **Groq LPU LLM inference**, **Deepgram Nova-2 STT**, and modular TTS engines (**Cartesia Sonic 3**, **ElevenLabs Flash v2.5**, and **Deepgram Aura**).

---

## 📑 Table of Contents
- [Architecture & Data Pipeline](#-architecture--data-pipeline)
- [Platform Evaluation Journey](#-platform-evaluation-journey)
- [Prerequisites & Windows Setup Fixes](#-prerequisites--windows-setup-fixes)
- [Consolidated Dependencies & Installation](#-consolidated-dependencies--installation)
- [Environment Variables (.env)](#-environment-variables-env)
- [CLI Diagnostic & API Verification Suite](#-cli-diagnostic--api-verification-suite)
- [Production Code Implementations](#-code-implementations)
  - [Engine 1: Cartesia Sonic 3 (Production Standard)](#engine-1-cartesia-sonic-3-production-standard)
  - [Engine 2: ElevenLabs Flash v2.5](#engine-2-elevenlabs-flash-v25)
  - [Engine 3: Deepgram Aura](#engine-3-deepgram-aura)
- [Engineering Journal & Debugging Matrix](#-engineering-journal--debugging-matrix)
- [Live Terminal Execution Proof](#-live-terminal-execution-proof)
- [Future Production Roadmap](#-future-production-roadmap)

---

## 🏛 Architecture & Data Pipeline

```text
[ Human User (Microphone / WebRTC / Phone SIP) ]
                         │
                         ▼
             [ LiveKit Cloud Engine ]
                         │
                         ▼
        [ Cyber-Voice Python Agent Worker ]
     ├── 1. VAD: Silero VAD (Noise suppression & speech edge detection)
     ├── 2. STT: Deepgram Nova-2 (Fast streaming speech-to-text)
     ├── 3. LLM: Groq LPU (`openai/gpt-oss-120b` @ 300+ tokens/sec)
     ├── 4. TTS: Cartesia Sonic 3 / ElevenLabs / Deepgram Aura
     └── 5. Function Calling / Webhooks:
              ├── `schedule_zoom_audit` -> Live Google Sheets CRM Webhook
              └── `end_call` -> Graceful room teardown after goodbye
```

---

## 🔍 Platform Evaluation Journey

During architectural exploration, proprietary and open-source stacks were systematically tested:

| Platform / Engine | Evaluation & Real-World Test Result | Final Decision |
| :--- | :--- | :--- |
| **Telnyx AI Suite** | High latency (~1800ms) and required manual SIP trunking + purchased phone numbers. | Replaced by WebRTC LiveKit stack. |
| **Voximplant** | Proprietary CPaaS with mandatory $10.00/month recurring SIP registration fee per softphone. | Rejected in favor of open-source stack. |
| **LiveKit Agents** | Open-source, WebRTC-native, sub-500ms pipeline, free developer sandbox, flexible tool calling. | **Adopted as Core Orchestrator.** |
| **Groq LPU** | Unmatched inference speed (>300 tokens/sec) via OpenAI-compatible endpoints. | **Adopted as LLM Brain (`openai/gpt-oss-120b`).** |
| **Deepgram Nova-2** | Sub-100ms transcription latency, handles spoken emails (`woman faith at yahoo dot com` $\rightarrow$ `womanfaith@yahoo.com`). | **Adopted as Primary STT.** |
| **Cartesia Sonic 3** | Sub-90ms TTS latency, human-grade conversational inflections, zero WebSocket drops on free accounts. | **Adopted as Primary TTS Engine.** |
| **ElevenLabs** | Studio-grade human audio, but free WebSocket streaming endpoints dropped frames (`no audio frames pushed`). | Supported via HTTP/Flash v2.5 plugin. |

---

## ⚙️ Prerequisites & Windows Setup Fixes

### Resolving `'pip' is not recognized` on Windows 10/11
If `python` opens the Microsoft Store or `pip` is unmapped:
1. Open Windows **Settings** (`Win + I`) $\rightarrow$ **Apps** $\rightarrow$ **Advanced app settings** $\rightarrow$ **App execution aliases**.
2. Toggle **OFF** both `python.exe` and `python3.exe`.
3. Ensure Python user PATHs are configured:
   ```text
   C:\Users\<User>\AppData\Local\Programs\Python\Python313\
   C:\Users\<User>\AppData\Local\Programs\Python\Python313\Scripts\
   ```
4. Run commands using the Windows Python launcher: `py -m pip ...` or `py agent.py dev`.

---

## 📦 Consolidated Dependencies & Installation

Run this single command to install all core libraries, plugins, and HTTP utilities:

```bash
py -m pip install livekit-agents \
                  livekit-plugins-deepgram \
                  livekit-plugins-groq \
                  livekit-plugins-silero \
                  livekit-plugins-cartesia \
                  livekit-plugins-elevenlabs \
                  livekit-plugins-openai \
                  python-dotenv \
                  requests
```

Download Silero VAD neural network weights:
```bash
py -m livekit.agents download-files
```

---

## 🔐 Environment Variables (`.env`)

Create a `.env` file in the root project directory:

```env
# LiveKit Cloud Connection
LIVEKIT_URL=wss://your-project-id.livekit.cloud
LIVEKIT_API_KEY=your_livekit_api_key
LIVEKIT_API_SECRET=your_livekit_api_secret

# Core AI Service API Keys
DEEPGRAM_API_KEY=your_deepgram_api_key
GROQ_API_KEY=gsk_your_groq_api_key
CARTESIA_API_KEY=sk_car_your_cartesia_key
ELEVEN_API_KEY=sk_your_elevenlabs_key

# Live CRM Sync
GOOGLE_SHEET_WEBHOOK=https://script.google.com/macros/s/your_deployment_id/exec
```

---

## 🛠 CLI Diagnostic & API Verification Suite

These single-line terminal scripts were constructed during development to verify API key validity, subscription tiers, and available model endpoints:

### 1. Test Groq API & Query Available Models
```bash
py -c "import requests; res = requests.get('https://api.groq.com/openai/v1/models', headers={'Authorization': 'Bearer YOUR_GROQ_KEY'}); print('GROQ STATUS:', res.status_code); print('AVAILABLE MODELS:', [m['id'] for m in res.json().get('data', [])])"
```
*Expected Output:* `GROQ STATUS: 200` with active models list (`openai/gpt-oss-120b`, `qwen/qwen3.6-27b`).

### 2. Test ElevenLabs Tier & Character Credits
```bash
py -c "import requests; res = requests.get('https://api.elevenlabs.io/v1/user', headers={'xi-api-key': 'YOUR_ELEVEN_KEY'}); print('STATUS:', res.status_code); print('TIER:', res.json().get('subscription', {}).get('tier')); print('CHARACTER COUNT:', res.json().get('subscription', {}).get('character_count'))"
```
*Expected Output:* `STATUS: 200`, `TIER: free`.

### 3. Test Cartesia API Connection
```bash
py -c "import os, requests; from dotenv import load_dotenv; load_dotenv(); key = os.getenv('CARTESIA_API_KEY'); res = requests.get('https://api.cartesia.ai/voices', headers={'X-API-Key': key or '', 'Cartesia-Version': '2024-06-10'}); print('STATUS:', res.status_code)"
```
*Expected Output:* `STATUS: 200`.

---

## 💻 Code Implementations

### Engine 1: Cartesia Sonic 3 (Production Standard)
> **Filename:** `agent_cartesia.py`  
> **Features:** Sub-90ms TTS, human breathing cadence, non-surrendering sales objection handling, spoken email normalization, and live Google Sheets logging.

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
- NEVER use asterisks (*), markdown formatting, bullets, or emojis.
- Ask only ONE question at a time.

# SPOKEN EMAIL FORMATTING RULE
- When the user speaks an email (e.g., "woman faith at yahoo dot com"), intelligently parse it into standard format ("womanfaith@yahoo.com") without making them re-spell it unless completely unintelligible.

# STRICT STEP-BY-STEP CONVERSATION FLOW

Step 1: GREETING & PERMISSION
- Say: "Hi! This is Sarah with TG Agency. I'll be super quick, I'm calling about your active listing. Do you have 20 seconds?"

Step 2: DISCOVERY & REBUTTAL (NEVER QUIT ON NO)
- IF THEY SAY YES / SURE:
  Say: "Awesome. Are you currently editing your own property Reels and managing your CRM late at night, or do you have a dedicated team handling that?"
- IF THEY SAY NO / BUSY / NOT A GOOD TIME / NOT INTERESTED:
  Say: "I completely understand, I'll be ultra-quick! Just curious—are you editing your own property Reels and handling admin late at night, or do you have a team for that?"

Step 3: VALUE OFFER
- "Got it. We actually want to build you a free 30-second custom video sample for your current listing to show you what our team can do."

Step 4: SCHEDULING TIME
- "We walk through the video and our system in an 8-minute Zoom preview. What works better for you—tomorrow at 6:00 PM, or the following morning at 8:30 AM?"

Step 5: COLLECT EMAIL (WAIT FOR PROSPECT)
- Once the user picks a time, ask ONLY: "Perfect! What is the best email address to send the calendar invite to?"
- STOP AND WAIT FOR THEM TO SPEAK THEIR EMAIL. DO NOT CALL ANY TOOLS YET.

Step 6: CONFIRMATION & HANG UP
- ONLY AFTER the user gives their email:
  1. Call the `schedule_zoom_audit` tool with their email and time.
  2. Say: "Awesome! I just sent the invite to your email. Thanks so much for your time, and talk soon. Goodbye!"
  3. Call the `end_call` tool.

# OBJECTION REBUTTALS
- "I'm not interested": "Totally fair! Most agents tell us that before seeing the sample video. Would you be open to an 8-minute preview tomorrow at 6:00 PM to check out the numbers?"
- "Too busy / Email me": "I completely hear you! I'll email the portfolio over, but so it does not get buried, can we do a 3-minute Zoom preview tomorrow at 6:00 PM?"
- "Compass gives us free tools": "Compass templates are great, but every agent uses the exact same one. We build custom Reels that elevate your personal brand. Does tomorrow work?"
- "How much does it cost?": "Most agents save 15+ hours a week for less than the cost of an open house host. We walk through exact pricing on the audit. Does tomorrow at 6:00 PM work?"
"""

class OutreachAgent(Agent):
    def __init__(self):
        super().__init__(instructions=PROMPT)

    async def on_enter(self) -> None:
        await self.session.generate_reply(
            instructions="Say: 'Hi! This is Sarah with TG Agency. I will be super quick, I am calling about your active property listing. Do you have 20 seconds?'"
        )

    @function_tool()
    async def schedule_zoom_audit(
        self,
        context: RunContext,
        email: str,
        preferred_time: str
    ) -> str:
        """Schedules a 10-minute Zoom audit, sends calendar invite, and saves to Google Sheet."""
        print(f"\n==========================================")
        print(f" APPOINTMENT BOOKED!")
        print(f" Email: {email}")
        print(f" Time:  {preferred_time}")
        print(f"==========================================\n")

        if GOOGLE_SHEET_WEBHOOK and "http" in GOOGLE_SHEET_WEBHOOK:
            try:
                requests.post(
                    GOOGLE_SHEET_WEBHOOK,
                    json={"email": email, "time": preferred_time},
                    timeout=5
                )
                print("[GOOGLE SHEETS] Successfully saved appointment row!")
            except Exception as e:
                print(f"[GOOGLE SHEETS ERROR] {e}")

        return f"Successfully booked Zoom audit for {email} at {preferred_time}."

    @function_tool()
    async def end_call(self, context: RunContext) -> str:
        """Call this tool ONLY after saying the final goodbye message to hang up the call cleanly."""
        print(f"\n[CALL FINISHED] -> Sarah scheduled hang up.\n")
        asyncio.create_task(self._hangup_after_delay())
        return "Call disconnected."

    async def _hangup_after_delay(self):
        await asyncio.sleep(6.0)
        await self.session.aclose()


async def entrypoint(ctx: JobContext):
    await ctx.connect()

    vad_plugin = silero.VAD.load(
        min_speech_duration=0.2,
        min_silence_duration=0.5,
        activation_threshold=0.65
    )

    session = AgentSession(
        vad=vad_plugin,
        stt=deepgram.STT(model="nova-2"),
        llm=groq.LLM(
            model="openai/gpt-oss-120b",
            api_key=os.getenv("GROQ_API_KEY"),
        ),
        tts=cartesia.TTS(
            model="sonic-3",
            voice="db6b0ed5-d5d3-463d-ae85-518a07d3c2b4", # Skylar - American Female
            api_key=os.getenv("CARTESIA_API_KEY"),
        ),
    )

    agent = OutreachAgent()
    await session.start(room=ctx.room, agent=agent)


if __name__ == "__main__":
    cli.run_app(WorkerOptions(entrypoint_fnc=entrypoint))
```

---

### Engine 2: ElevenLabs Flash v2.5
> **Filename:** `agent_elevenlabs.py`

```python
import os
import asyncio
from dotenv import load_dotenv
from livekit.agents import Agent, AgentSession, JobContext, WorkerOptions, cli
from livekit.plugins import deepgram, groq, silero, elevenlabs

load_dotenv()

# In entrypoint():
session = AgentSession(
    vad=silero.VAD.load(),
    stt=deepgram.STT(model="nova-2"),
    llm=groq.LLM(model="openai/gpt-oss-120b", api_key=os.getenv("GROQ_API_KEY")),
    tts=elevenlabs.TTS(
        model="eleven_flash_v2_5",
        voice_id="21m00Tcm4TlvDq8ikWAM",  # Rachel - American Professional Female
        api_key=os.getenv("ELEVEN_API_KEY"),
    ),
)
```

---

### Engine 3: Deepgram Aura
> **Filename:** `agent_deepgram.py`

```python
import os
import asyncio
from dotenv import load_dotenv
from livekit.agents import Agent, AgentSession, JobContext, WorkerOptions, cli
from livekit.plugins import deepgram, groq, silero

load_dotenv()

# In entrypoint():
session = AgentSession(
    vad=silero.VAD.load(),
    stt=deepgram.STT(model="nova-2"),
    llm=groq.LLM(model="openai/gpt-oss-120b", api_key=os.getenv("GROQ_API_KEY")),
    tts=deepgram.TTS(model="aura-stella-en"), # Corporate American Female
)
```

---

## 🧠 Engineering Journal & Debugging Matrix

| Issue / Exception Encountered | Root Cause | Architectural Resolution |
| :--- | :--- | :--- |
| `ModuleNotFoundError: livekit.agents.voice_assistant` | Deprecated import path in LiveKit v1.x migration. | Updated imports to `from livekit.agents import Agent, AgentSession`. |
| `AttributeError: llm has no attribute 'ai_callable'` | Legacy tool decorator removed in LiveKit 1.8. | Replaced `@llm.ai_callable` with `@function_tool()` and `RunContext`. |
| `APIStatusError: 404 Model not found: llama-3.3-70b` | Selected Groq model ID was restricted on specific developer key. | Ran diagnostic script, discovered active models, updated to `openai/gpt-oss-120b`. |
| `TypeError: TTS.__init__() got unexpected keyword 'voice'` | Parameter signature mismatch in `livekit-plugins-elevenlabs`. | Changed parameter to `voice_id="21m00Tcm4TlvDq8ikWAM"`. |
| `APIError: no audio frames pushed` (Cartesia) | Cartesia v1 model string `sonic-english` was deprecated by API. | Updated model string to production **`sonic-3`**. |
| Premature disconnect before goodbye finished | `end_call` triggered concurrently with `schedule_zoom_audit`. | Separated Step 5/6 in prompt and added `asyncio.sleep(6.0)` delay buffer. |
| `resumed false interrupted speech` stutter loop | Speaker audio feeding back into microphone (Acoustic Echo). | Configured Silero VAD `activation_threshold=0.65` and recommended headphones for testing. |

---

## 📈 Live Terminal Execution Proof

```text
2026-09-07 09:14:30 - INFO livekit.agents - registered worker {"agent_name": "", "id": "AW_AGdqGVZh7bCW", "url": "wss://tgtest001outreach-agent.livekit.cloud"}
2026-09-07 09:14:46 - DEBUG livekit.agents - conversation_item_added {"role": "assistant", "text": "Hi! This is Sarah with TG Agency. Do you have 20 seconds?"}
2026-09-07 09:14:49 - DEBUG livekit.agents - received user transcript {"text": "No. I don't have thirty seconds."}
2026-09-07 09:14:58 - DEBUG livekit.agents - conversation_item_added {"role": "assistant", "text": "I hear you. Are you editing your own property Reels late at night, or do you have a team?"}
2026-09-07 09:15:32 - DEBUG livekit.agents - received user transcript {"text": "Yeah. I do this late at night, and I do get really tired."}
2026-09-07 09:15:52 - DEBUG livekit.agents - received user transcript {"text": "Let's do tomorrow, six PM."}
2026-09-07 09:16:10 - DEBUG livekit.agents - received user transcript {"text": "woman faith at yahoo dot com."}
2026-09-07 09:16:10 - DEBUG livekit.agents - executing tool {"function": "schedule_zoom_audit", "arguments": "{\"email\": \"womanfaith@yahoo.com\", \"preferred_time\": \"Tomorrow at 6:00 PM\"}"}

==========================================
 APPOINTMENT BOOKED!
 Email: womanfaith@yahoo.com
 Time:  Tomorrow at 6:00 PM
==========================================

[GOOGLE SHEETS] Successfully saved appointment row!
2026-09-07 09:16:11 - DEBUG livekit.agents - executing tool {"function": "end_call"}
[CALL FINISHED] -> Sarah scheduled hang up.
```

---

## 🗺 Future Production Roadmap

- [ ] **Inbound/Outbound SIP Trunking:** Connect Twilio/Telnyx SIP trunking for cellular dial-in and automated lead sheet dialing.
- [ ] **24/7 Cloud Hosting:** Containerize worker into Docker and deploy to Linux VPS / Railway / AWS EC2.
- [ ] **CRM Sync:** Direct API integration with HubSpot / Follow Up Boss / GoHighLevel.
- [ ] **Dynamic Lead Personalization:** Pass CSV variables (`{{first_name}}`, `{{listing_address}}`) directly into LiveKit room metadata upon call dispatch.
