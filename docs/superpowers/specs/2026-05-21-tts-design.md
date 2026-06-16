# TTS Feature Design — AI Narrative System
**Date:** 2026-05-21

## Overview
Add Text-to-Speech to the narrative frontend using Edge TTS (backend) so each character speaks their dialogue aloud. Text is Roman Urdu (Latin script), so English neural voices are used.

## Architecture

```
Frontend (React)              Backend (FastAPI)
─────────────────             ─────────────────
Turn renders
  ↓
Speaker icon clicked      →   GET /api/tts?text=...&voice=...
Auto-play fires           →       edge-tts generates MP3 in memory
                          ←   StreamingResponse(audio/mpeg)
<audio> element plays MP3
Previous audio stops
```

## Backend

**New endpoint:** `GET /api/tts`

- Query params: `text` (str), `voice` (str, optional — defaults to Aria)
- Uses `edge-tts` Python package (no API key, free)
- Generates audio in memory (BytesIO), streams as `audio/mpeg`
- Added to `src/api.py`

**Dependency:** `uv add edge-tts`

## Character → Voice Mapping

| Character | Voice |
|-----------|-------|
| Saleem | `en-US-GuyNeural` |
| Ahmed Malik | `en-US-DavisNeural` |
| Constable Raza | `en-US-TonyNeural` |
| Uncle Jameel | `en-US-JasonNeural` |
| Narration (Director) | `en-US-AriaNeural` |

Mapping lives in frontend `App.jsx` as a const — easy to update.

## Frontend Changes (`App.jsx`)

**Manual mode:**
- Speaker icon button on each turn card (Volume2, already imported)
- Click → fetch `/api/tts` → play via `<audio>` element
- Button shows loading spinner while fetching
- Button pulses while playing
- Clicking again stops audio

**Auto-play mode:**
- TTS auto-triggers when turn changes
- Previous audio ref cancelled before new fetch
- 8s auto-advance timer starts AFTER TTS completes (or on error)

**State additions:**
- `isSpeaking` (bool) — audio currently playing
- `audioRef` (useRef) — current Audio object, for stopping

## Error Handling

- TTS fetch fails → silent fail, story continues normally
- User skips turn mid-audio → `audioRef.current.pause()` called immediately
- Duplicate request guard — ignore click if already fetching/playing

## Out of Scope
- Urdu script support (text is Roman Urdu, English voices handle it)
- Voice selection UI (fixed per character)
- Caching TTS audio (each run generates fresh)
