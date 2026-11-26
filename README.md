# AI Story Video Generator

An end-to-end automated storytelling and video generation pipeline that transforms a text idea into a narrated, subtitled multi-scene video using **FastAPI**, **Next.js**, **FFmpeg**, **Edge TTS**, **Whisper**, and **Hugging Face ZeroGPU Spaces**.

---

## Architecture

```text
[Browser / Next.js 14 Frontend]
        │
        │  HTTP / JSON
        ▼
[FastAPI Orchestrator (Python)]
        ├─ LLM Script & Scene Generator ──► scripts.json
        ├─ Edge TTS Narration Engine    ──► audio.mp3
        ├─ Text-to-Image (T2I Space)    ──► image1.png, image2.png
        ├─ Image-to-Video (I2V Space)   ──► vid1.mp4, vid2.mp4
        ├─ FFmpeg Composition & Concat  ──► seg_XX.mp4, final_output.mp4
        └─ Whisper + ImageMagick Burn-in──► final_output_subtitled.mp4
```

Session artifacts are persisted under `sessions/<SESSION_ID>/` with deterministic duration alignment between narration audio and generated video segments, plus automatic retry/backoff handling for inference requests.

---

## Key Features

- **End-to-End Media Pipeline:** Idea → multi-scene scripts → scene images → animated video clips → stitched timeline → Whisper-aligned subtitled MP4.
- **ZeroGPU Cloud Inference:** Offloads heavy Text-to-Image (Qwen-Image) and Image-to-Video (Wan 2.2 14B) inference to Hugging Face Spaces; no local GPU required.
- **Resilient Orchestration:** Built-in retries/backoff for I2V generation, local fallback for T2I, and deterministic audio-to-video duration synchronization.
- **Session & Artifact APIs:** RESTful endpoints for session creation, script generation, pipeline execution, status polling, and MP4 downloads.
- **Modern Frontend:** Next.js 14 UI for managing sessions, selecting generated scripts/voices, monitoring pipeline progress, and downloading final renders.

---

## Tech Stack

- **Backend & Orchestration:** Python 3.10+, FastAPI, Uvicorn
- **Media Processing:** FFmpeg, MoviePy, ImageMagick, OpenAI Whisper, Edge TTS
- **AI Inference:** Groq / LLM APIs, Hugging Face Gradio Client (T2I & I2V Spaces)
- **Frontend:** Next.js 14, TypeScript, React

---

## Setup & Usage

### 1. Environment Variables
Create a `.env` file in the project root:

```env
GROQ_API_KEY=your_groq_api_key
HF_TOKEN=your_huggingface_token
HF_TOKEN_VIDEO=your_huggingface_video_token

# Optional runtime settings
EDGE_TTS_VOICE=en-US-AriaNeural
WHISPER_MODEL_SIZE=base
WHISPER_LANG=en
SUBTITLES_FONT=Segoe UI
VIDEO_GEN_WAIT_SECONDS=10
VIDEO_GEN_MAX_RETRIES=5
```

### 2. Backend (FastAPI)
Ensure **FFmpeg** and **ImageMagick** are installed and available on your `PATH`.

```powershell
py -3.11 -m venv .venv
. .\.venv\Scripts\Activate.ps1
pip install -r requirements.txt

uvicorn api.server:app --host 0.0.0.0 --port 8000 --reload
```

### 3. Frontend (Next.js)
```powershell
cd frontend
pnpm install
pnpm dev
```

The frontend connects to `http://localhost:8000` by default (configurable in `frontend/lib/api.ts`).

---

## API Reference (`api/server.py`)

| Method | Endpoint | Description |
| :--- | :--- | :--- |
| `POST` | `/api/sessions` | Create a new story generation session (`{ "idea": "..." }`) |
| `POST` | `/api/scripts` | Generate multi-scene scripts for a session |
| `POST` | `/api/sessions/{session_id}/run` | Execute the pipeline (`script_key`, `voice`, `mode`: `"videos"` or `"images"`) |
| `GET` | `/api/sessions` | List all sessions |
| `GET` | `/api/sessions/{session_id}` | Poll pipeline status, progress, and artifact URLs |
| `GET` | `/api/sessions/{session_id}/download/{final\|subtitled}` | Download final or subtitled MP4 |
| `GET` | `/api/sessions/{session_id}/artifact/{relpath}` | Fetch any individual session artifact |
| `GET` | `/api/voices` | List available Edge TTS voices (`?flat=true` for flat list) |

---

## Session Artifact Structure

```text
sessions/<SESSION_ID>/
  idea.txt
  scripts.json
  status.json
  segments/seg_XX/
    segment.txt
    audio.mp3
    image1.png
    image2.png
    vid1.mp4
    vid2.mp4
    seg_XX.mp4
  final_output.mp4
  final_output_subtitled.mp4
```
