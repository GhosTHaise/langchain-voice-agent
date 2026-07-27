# LangChain Voice Agent

A real-time voice assistant built with LangChain, Gemini, AssemblyAI, and
Cartesia. Speak in the browser, watch each stage of the pipeline, and hear the
agent respond.

The included demo behaves like a sandwich-shop assistant. It keeps
conversation state, calls tools to build and confirm an order, and exposes
transcription, tool activity, response, and latency data in a Svelte dashboard.

> [!NOTE]
> This project is an experimental reference implementation. It is best suited
> to local development and prototyping, not production deployment as-is.

## What it includes

- Live microphone capture in the browser at 16 kHz PCM
- Partial and final transcription with AssemblyAI
- A stateful LangChain agent powered by Gemini
- Observable tool calls and results
- Cartesia speech synthesis with streamed PCM playback
- A Svelte 5 dashboard with pipeline status, activity, logs, and latency
- A WebSocket transport shared by audio input and pipeline events

## Architecture

```text
Browser microphone
       │  PCM audio over WebSocket
       ▼
FastAPI ──► AssemblyAI STT
       │          │
       │          ▼
       │     LangChain agent ──► tools
       │          │
       │          ▼
       │      Cartesia TTS
       │          │
       ◄──────────┘  JSON events + base64 PCM audio
       │
       ▼
Svelte dashboard and browser audio playback
```

The backend serves the compiled frontend at `/` and handles the live session
at `/ws`. Pipeline stages communicate through typed events such as
`stt_chunk`, `stt_output`, `agent_chunk`, `tool_call`, `tool_result`, and
`tts_chunk`.

## Prerequisites

- Python 3.12
- [uv](https://docs.astral.sh/uv/)
- Node.js 20.19+ or 22.12+
- [pnpm](https://pnpm.io/)
- API keys for AssemblyAI, Cartesia, and Google Gemini
- A browser with microphone access

## Quick start

1. Clone the repository.

   ```bash
   git clone https://github.com/GhosTHaise/langchain-voice-agent.git
   cd langchain-voice-agent
   ```

2. Install the Python and frontend dependencies.

   ```bash
   uv sync
   pnpm --dir web install
   ```

3. Create a `.env` file in the repository root.

   ```dotenv
   ASSEMBLYAI_API_KEY=your_assemblyai_api_key
   CARTESIA_API_KEY=your_cartesia_api_key
   GOOGLE_API_KEY=your_google_api_key
   ```

4. Build the frontend and start the application.

   ```bash
   pnpm --dir web build
   uv run python src/main.py
   ```

5. Open [http://localhost:8000](http://localhost:8000), allow microphone
   access, and start a session.

The `.env` file is ignored by Git. Keep API keys local and never commit them.

## Development

Run the backend and Vite development server in separate terminals:

```bash
# Terminal 1
uv run python src/main.py
```

```bash
# Terminal 2
pnpm --dir web dev
```

Then open [http://localhost:5173](http://localhost:5173). Vite proxies `/ws`
to the backend on port 8000.

The backend still expects `web/dist` to exist on startup. Run
`pnpm --dir web build` once before starting it, and rebuild whenever you want
the version served on port 8000 to reflect frontend changes.

Useful checks:

```bash
uv run python -m compileall -q src
pnpm --dir web check
pnpm --dir web build
```

## Configuration

The demo currently keeps configuration close to the implementation:

| Setting | Location | Current value |
| --- | --- | --- |
| Agent model and tools | `src/main.py` | `gemini-2.5-flash-lite` |
| Agent behavior | `src/main.py` | Sandwich-shop assistant |
| STT sample rate | `src/main.py`, `src/assemblyai_stt.py` | 16 kHz |
| Cartesia model and voice | `src/cartesia_tts.py` | `sonic-3`, preset voice ID |
| TTS output | `src/cartesia_tts.py` | 24 kHz, 16-bit PCM |
| Frontend WebSocket client | `web/src/lib/websocket.ts` | Same host, `/ws` |

To adapt the agent, edit `system_prompt`, replace the example order tools, and
update the `tools` list passed to `create_agent` in `src/main.py`.

An ElevenLabs TTS client is also available in `src/elevenlabs_tts.py`, but the
active pipeline uses Cartesia. Switching providers requires wiring that client
into the TTS stage and setting `ELEVENLABS_API_KEY`.

## Project structure

```text
.
├── src/
│   ├── main.py              # FastAPI app and voice pipeline
│   ├── assemblyai_stt.py    # Streaming speech-to-text client
│   ├── cartesia_tts.py      # Active text-to-speech client
│   ├── elevenlabs_tts.py    # Alternative text-to-speech client
│   ├── events.py            # Typed pipeline event definitions
│   └── utils.py             # Async stream helpers
├── web/
│   └── src/
│       ├── lib/audio/       # Browser capture and playback
│       ├── lib/components/  # Dashboard components
│       └── lib/stores/      # Session and latency state
├── pyproject.toml
└── uv.lock
```

## Troubleshooting

**`Web build not found`**

Build the Svelte app before starting the backend:

```bash
pnpm --dir web build
```

**Microphone access is denied**

Allow microphone access in the browser. Outside localhost, browser audio
capture generally requires HTTPS.

**The session connects but no transcript or audio appears**

Confirm all three API keys are present in the root `.env` file, restart the
backend after changing it, and inspect the dashboard console plus the backend
terminal for provider errors.

**Audio playback is choppy**

Use headphones to reduce echo, close other microphone-heavy applications, and
check the network path to the transcription and speech providers.

## Current limitations

- Conversation memory is in-process and is lost when the server restarts.
- CORS is open and the WebSocket endpoint has no authentication.
- One WebSocket connection owns one in-memory conversation session.
- The frontend must be built before the backend can start.
- Provider retries, rate-limit handling, persistence, and production
  observability are not implemented.
