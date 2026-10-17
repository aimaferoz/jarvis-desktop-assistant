# JARVIS — Clap-Activated Desktop Automation Assistant

A Python-based **Windows desktop automation prototype** that turns a double-clap gesture into a personalized workspace setup. JARVIS listens to microphone input, detects a pair of short audio transients, and runs a predefined sequence of desktop actions.

> **Project status:** Developed and used on Windows by its creator. The publication package passes a Python syntax check; functionality has not been independently retested on another Windows machine.

## Features

- **Double-clap recognition:** Processes live microphone audio with NumPy RMS measurements, an adaptive noise floor, timing windows, and retrigger/debounce logic.
- **Microphone selection:** Probes the default Windows input and can fall back to another available microphone.
- **Workspace automation:** Opens a Spotify track, Claude and Binance pages in Chrome, and focuses or launches Cursor IDE.
- **Multi-monitor support:** Uses Windows APIs to position Chrome windows on configured monitors and optionally enter fullscreen.
- **Voice welcome:** Generates a spoken greeting through ElevenLabs text-to-speech.
- **Speech caching:** Stores generated WAV audio locally so the same greeting can be replayed without another API request.

**Tech stack:** Python, NumPy, SoundDevice/PortAudio, ElevenLabs, python-dotenv, Windows APIs (`ctypes`), Chrome, Cursor.

This is a **gesture-triggered automation assistant**, not a conversational or voice-command assistant.

## How it works

```text
Microphone input
      |
      v
RMS audio level + adaptive noise threshold
      |
      v
Two valid transients within the clap time window?
      |
      +-- No --> Continue listening
      |
      +-- Yes --> Run configured actions in background
                       |-- Open Spotify track
                       |-- Open Chrome pages / position windows
                       |-- Play ElevenLabs greeting (or cached WAV)
                       `-- Focus / launch Cursor
```

The welcome sequence runs **once per script process**; restart JARVIS to trigger it again.

## Requirements

- Windows (Windows-specific window management requires Windows)
- Python 3.10+ recommended
- A working microphone and microphone permissions
- Google Chrome and Cursor for the corresponding optional actions
- An ElevenLabs account/API key for generated welcome speech

## Installation

1. Download or clone the repository and open its folder in a terminal.
2. Install dependencies:

   ```bash
   python -m pip install -r requirements.txt
   ```

3. Copy `.env.example` to `.env`, then insert your own ElevenLabs credentials. **Never commit `.env`.**
4. Review the constants near the top of `jarvis.py` for your Spotify link, browser actions, and monitor arrangement.
5. Start the listener:

   ```bash
   python jarvis.py
   ```

6. Clap twice near the microphone. Stop the listener with **Ctrl+C**.

## Configuration

### Environment variables

| Variable | Purpose |
| --- | --- |
| `ELEVENLABS_API_KEY` | Private API key for ElevenLabs text-to-speech |
| `ELEVENLABS_VOICE_ID` | ElevenLabs voice identifier |
| `ELEVENLABS_MODEL_ID` | Optional; default `eleven_multilingual_v2` |
| `ELEVENLABS_OUTPUT_FORMAT` | Optional; default `pcm_24000` |
| `ELEVENLABS_PCM_SAMPLE_RATE` | Optional PCM sample-rate override |
| `JARVIS_INPUT_DEVICE` | Optional microphone index or partial device name |
| `JARVIS_WELCOME_CACHE_DIR` | Optional WAV cache folder override |
| `CLAUDE_CODE_URL` | Optional Chrome destination for Claude |
| `BINANCE_BTC_URL` | Optional Chrome destination for Binance |
| `CHROME_NEW_WINDOW_WAIT_S` | Optional Chrome window wait timeout |
| `CHROME_WINDOW_WIDTH`, `CHROME_WINDOW_HEIGHT` | Optional Chrome window dimensions |

The `.env` file is loaded from the same folder as `jarvis.py`. Without ElevenLabs credentials, the speech step is skipped, but other actions can still run.

### Constants in `jarvis.py`

| Constant | Purpose |
| --- | --- |
| `SAMPLE_RATE` | Microphone sample rate (default 44100 Hz) |
| `BLOCK_MS` | Audio analysis block length |
| `SPIKE_RATIO` | Relative loudness threshold for a clap |
| `MIN_RMS` | Absolute minimum clap level |
| `MIN_DOUBLE_GAP_S`, `MAX_DOUBLE_GAP_S` | Allowed time between claps |
| `COOLDOWN_S` | Debounce interval |
| `SONG_URI` | Track opened after a double clap |
| `OPEN_CLAUDE_CODE_IN_CHROME`, `OPEN_BINANCE_BTC_IN_CHROME` | Browser action toggles |
| `CLAUDE_CHROME_MONITOR`, `BINANCE_CHROME_MONITOR` | Monitor indices (1-based) |
| `FOCUS_EXISTING_CURSOR_ON_DOUBLE_CLAP`, `OPEN_NEW_CURSOR_ON_DOUBLE_CLAP` | Cursor actions |
| `CURSOR_OPEN_FULLSCREEN`, `OPEN_CHROME_FULLSCREEN` | Fullscreen behavior |
| `JARVIS_WELCOME_ENABLED`, `JARVIS_WELCOME_PHRASE` | Speech greeting controls |

The repository preserves the creator's original working defaults. Customize these settings to suit your own machine before running it.

## Troubleshooting

- **No clap detected:** Confirm microphone permissions and input levels. Try a lower `SPIKE_RATIO` or clap nearer the microphone.
- **Too many false detections:** Increase `SPIKE_RATIO` or `MIN_RMS`.
- **Wrong microphone:** Set `JARVIS_INPUT_DEVICE` in `.env`. List inputs using `python -c "import sounddevice as sd; print(sd.query_devices())"`.
- **PortAudio error:** Check microphone drivers and try `SAMPLE_RATE = 48000`.
- **Chrome opens on an unexpected screen:** Adjust the 1-based monitor constants to match your display arrangement.
- **No welcome speech:** Check the ElevenLabs key/voice ID and API access; cached audio is saved under `.cache/jarvis_welcome/` by default.

## Security

Keep your `.env` and API keys private. The included `.env.example` contains placeholders only. `.gitignore` excludes local secrets, Python environments, and generated audio caches. If an API key has ever been shared or committed, **revoke and rotate it** before publishing.

## Future improvements

Potential next steps include a settings file for user-specific preferences, configurable action profiles, and automated tests for clap-detection logic.
