# Pyrexia — AI Clinic Triage

> An AI-powered, privacy-first medical triage assistant that runs entirely in your browser using a local language model. No patient data ever leaves the device.

---

## Overview

Pyrexia is a single-file web application designed to assist clinic staff with patient triage. It guides patients through a structured symptom-collection conversation, assigns a real-time emergency score, and updates a live patient queue — all powered by a locally running LLM via [Ollama](https://ollama.com/).

---

## Features

### Patient Intake
- Structured registration form capturing name, age, sex, known medical conditions, and presenting symptoms.
- Data is held in memory for the session only; nothing is persisted or transmitted externally.

### AI-Powered Symptom Assessment
- Conversational triage conducted by an AI assistant named *Pyrexia*, backed by **llama3.1** running locally via Ollama.
- The assistant asks one focused question at a time, covering onset, severity, location, duration, and associated symptoms.
- Responds exclusively in the patient's selected language.
- Will immediately advise emergency contact if life-threatening symptoms are detected (chest pain, stroke signs, anaphylaxis, etc.).

### Emergency Scoring Engine
- After two or more conversation turns the model assigns a **0–100 emergency score** and a triage category:

  | Score | Category | Action |
  |-------|----------|--------|
  | 0 – 39 | **Basic** | Standard queue |
  | 40 – 79 | **Attention** | Priority queue |
  | 80 – 100 | **Critical** | Immediate — emergency overlay triggered |

- Score is extracted via a secondary LLM prompt that returns structured JSON; the UI gauge and queue position update automatically.

### Critical Mode
- When a score ≥ 80 is detected, the UI enters *critical mode*: full-screen flashing red overlay, animated alert, audible voice announcement, and the page background pulses.
- Medical staff can dismiss the overlay to confirm the team has been notified.

### Patient Queue / Leaderboard
- A live sidebar table lists all assessed patients in the current session, sorted by emergency score descending.
- New patients are highlighted on entry; score badges are colour-coded (green / amber / red).

### Multilingual Support
- Voice input and text-to-speech are matched to the selected language.
- Supported languages: English (en-IN), Hindi, Tamil, Telugu, Kannada, Malayalam, Marathi, Bengali, Spanish, French, Arabic, Chinese, German.

### Voice Input
- One-click microphone recording via the **Web Speech API**.
- Continuous recognition with interim transcription displayed in real time.
- Automatic send triggered by a configurable silence timeout (default 2.2 s), with a visual countdown bar.
- Voice button pulses red while recording; manual stop is also supported.

### Text-to-Speech Output
- Every AI response is spoken aloud using the browser's **SpeechSynthesis API** in the patient's chosen language, aiding low-literacy users.

---

## Prerequisites

| Requirement | Notes |
|-------------|-------|
| [Ollama](https://ollama.com/) | Must be running locally on `http://localhost:11434` |
| **llama3.1** model | Pull with `ollama pull llama3.1` |
| **Google Chrome** (recommended) | Required for Web Speech API (voice input). Other Chromium-based browsers may also work. |
| Internet connection (initial load) | Google Fonts are loaded from CDN; the rest runs fully offline. |

---

## Getting Started

**1. Install and start Ollama**

```bash
# macOS / Linux
curl -fsSL https://ollama.com/install.sh | sh
ollama serve
```

**2. Pull the model**

```bash
ollama pull llama3.1
```

**3. Open the app**

Simply open `pyrexia.html` in Google Chrome — no build step, no server, no dependencies to install.

```bash
open pyrexia.html   # macOS
xdg-open pyrexia.html  # Linux
```

> The header status indicator will turn green once the app confirms Ollama is reachable.

---

## Usage

1. **Register the patient** — complete the intake form (name, age, sex, known conditions, primary symptom) and click **Begin Triage Assessment**.
2. **Select a language** — choose from the language bar above the chat input, or speak and the transcript will follow the selected locale.
3. **Converse** — type or use the microphone button to describe symptoms. The AI will ask clarifying questions.
4. **Monitor the score** — the Emergency Score gauge and queue position update live in the sidebar after each exchange.
5. **Critical alerts** — if the score reaches 80+, acknowledge the overlay and escalate to clinical staff immediately.

---

## Configuration

The following constants at the top of the `<script>` block can be adjusted:

| Constant | Default | Description |
|----------|---------|-------------|
| `OLLAMA_URL` | `http://localhost:11434/api/generate` | Ollama API endpoint |
| `MODEL` | `llama3.1` | Ollama model name |
| `STATE.SILENCE_TIMEOUT` | `2200` (ms) | Silence duration before voice auto-sends |

---

## Architecture

Pyrexia is a **zero-dependency, single HTML file** application.

```
pyrexia.html
├── CSS           — Custom design system with CSS variables, dark theme, responsive layout
├── HTML          — Two-page SPA (intake form → chat interface)
└── JavaScript
    ├── State management     — Plain JS object (STATE)
    ├── Ollama integration   — fetch() calls to local REST API
    ├── Score extraction     — Secondary LLM prompt → JSON parse
    ├── Web Speech API       — Voice input with silence detection
    ├── SpeechSynthesis API  — Text-to-speech output
    └── Patient queue        — In-memory leaderboard with live re-render
```

---

## Privacy & Safety

- **All AI processing is local.** Ollama runs on-device; no patient data is sent to external servers.
- **No persistence.** Patient data exists only in memory for the duration of the browser session.
- **Not a diagnostic tool.** Pyrexia collects and summarises symptoms for qualified medical staff. It does not diagnose, prescribe, or replace a licensed physician's assessment.

---

## Browser Compatibility

| Feature | Chrome | Edge | Firefox | Safari |
|---------|--------|------|---------|--------|
| Core chat | ✅ | ✅ | ✅ | ✅ |
| Voice input | ✅ | ✅ | ❌ | ⚠️ Partial |
| Text-to-speech | ✅ | ✅ | ✅ | ✅ |

Voice input (`webkitSpeechRecognition`) requires a Chromium-based browser.

---

## License

This project is provided for educational and prototyping purposes. It is not certified for clinical use. Consult your local regulations before deploying any AI tool in a healthcare setting.
