# JARVIS — Double-Clap Desktop Assistant

A Python desktop automation prototype that detects a **double clap** through microphone audio and runs a configurable welcome workflow. It combines audio signal processing, Windows application control, browser window management, and optional ElevenLabs text-to-speech.

> **Status:** Portfolio prototype. The Python source passes a syntax check, but the full workflow has **not** been independently tested on Windows as part of this publication package.

## What it does

- Reads microphone audio using `sounddevice` and computes RMS levels with NumPy.
- Detects two audio transients within a configurable time window using an adaptive noise floor, retrigger threshold, and cooldown.
- On the first detected double clap, launches configured desktop actions in a background thread.
- Can focus or open Cursor, open a music link, and optionally launch Chrome pages on selected monitors.
- Optionally generates a welcome message with ElevenLabs and caches the audio locally.
- Supports microphone selection through the `JARVIS_INPUT_DEVICE` environment variable.

**Important:** This version is clap-activated, not a conversational speech-recognition or spoken-command assistant. The welcome sequence is intentionally limited to once per run.

## Requirements

- Python 3.10+ recommended
- Microphone with permission enabled
- Windows for the full Cursor/window-management experience
- Optional: Cursor, Google Chrome, and ElevenLabs API credentials

## Getting started

```bash
python -m venv .venv
# Windows PowerShell:
.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
```

Optional voice output: copy `.env.example` to `.env` and enter your own ElevenLabs credentials. **Never commit `.env`.** If no voice ID is set, the script skips the welcome audio.

Run:

```bash
python jarvis.py
```

Clap twice near the microphone. Stop the listener with `Ctrl+C`.

## Configuration

Edit the constants near the top of `jarvis.py`:

| Setting | Purpose |
| --- | --- |
| `SPIKE_RATIO`, `MIN_RMS` | Audio transient sensitivity |
| `MIN_DOUBLE_GAP_S`, `MAX_DOUBLE_GAP_S` | Allowed interval between claps |
| `COOLDOWN_S`, `RETRIGGER_RATIO` | Debouncing and re-arming |
| `SONG_URI` | Optional music URL; blank by default |
| `FOCUS_EXISTING_CURSOR_ON_DOUBLE_CLAP` | Focus or launch Cursor |
| `OPEN_CLAUDE_CODE_IN_CHROME` | Opt-in browser launch (off by default) |
| `OPEN_BINANCE_BTC_IN_CHROME` | Opt-in browser launch (off by default) |
| `CLAUDE_CHROME_MONITOR`, `BINANCE_CHROME_MONITOR` | Windows display selection |
| `JARVIS_WELCOME_PHRASE` | Text spoken by ElevenLabs |

The sample defaults do not automatically open personal browsing pages or music links. Enable those actions only if you want them.

## Technical design

`Microphone → audio blocks → RMS and adaptive noise floor → double-clap detector → background workflow → applications / optional speech`

The program includes Windows-specific APIs for focusing applications and positioning browser windows. Behavior on other operating systems is limited.

## Troubleshooting

- **No clap detection:** Confirm Windows microphone permissions and check the device list printed at startup. Try lowering `SPIKE_RATIO` slightly.
- **False detections:** Raise `SPIKE_RATIO` or `MIN_RMS`.
- **No welcome speech:** Configure `ELEVENLABS_API_KEY` and `ELEVENLABS_VOICE_ID` in `.env` and check your API access.
- **Cursor doesn't launch:** Install Cursor or add its executable to your PATH.
- **Wrong monitor:** Adjust the 1-based monitor constants near the top of the script.

## Project structure

```text
jarvis-desktop-assistant/
├── jarvis.py
├── requirements.txt
├── README.md
├── .env.example
└── .gitignore
```

## Security and publication notes

No credentials are included in this repository. Keep your actual `.env` local. Before making the repository public, test the program on your own machine and review the code for personal settings and unintended application launches.
