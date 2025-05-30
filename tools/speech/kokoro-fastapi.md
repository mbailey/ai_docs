# Kokoro-FastAPI Documentation

## Overview
Kokoro-FastAPI is a FastAPI-based server that provides an OpenAI-compatible TTS API for the Kokoro-82M text-to-speech model. It supports multiple languages (English, Japanese, Chinese) and offers both GPU and CPU inference.

## Key Features
- **OpenAI-compatible API**: Drop-in replacement for OpenAI's TTS endpoint
- **Multiple voices**: 70+ voice presets across different languages and styles
- **Flexible deployment**: Docker containers for both CPU and GPU
- **Web UI**: Built-in web interface at localhost:8880/web
- **Advanced features**: Phoneme generation, word timestamps, voice mixing

## Architecture
- FastAPI server (main.py)
- Model manager for loading Kokoro TTS model
- Voice manager with 70+ voice embeddings
- OpenAI-compatible router at `/v1/audio/speech`
- Debug endpoints for monitoring
- Web player interface

## Integration Points for voice-mcp

### 1. API Endpoint
- Base URL: `http://localhost:8880`
- OpenAI endpoint: `/v1/audio/speech`
- Accepts same parameters as OpenAI TTS

### 2. Available Voices
Some notable voices compatible with OpenAI naming:
- `nova` (af_nova.pt) - Female voice
- `alloy` (af_alloy.pt) - Female voice  
- `echo` (am_echo.pt) - Male voice
- `onyx` (am_onyx.pt) - Male voice
- Many more language-specific options

### 3. Container Options
- **CPU Image**: `ghcr.io/remsky/kokoro-fastapi-cpu:latest`
- **GPU Image**: `ghcr.io/remsky/kokoro-fastapi-gpu:latest`
- **Local Build**: Can build from Dockerfile in docker/cpu or docker/gpu

### 4. Environment Variables
Key settings from core/config.py:
- `PYTHONPATH=/app:/app/api`
- `ONNX_NUM_THREADS` (CPU optimization)
- Can override API host/port

## Benefits for voice-mcp
1. **Local TTS**: No OpenAI API costs
2. **Privacy**: Audio generation stays local
3. **Customization**: 70+ voice options vs OpenAI's limited set
4. **Latency**: Potentially lower latency for local deployments
5. **Compatibility**: Drop-in replacement for existing OpenAI TTS code

## Usage

### Web Interface
- Available at: http://localhost:8880/web/
- **Browser Compatibility**: 
  - ✅ Chrome/Chromium (full streaming support)
  - ❌ Firefox (MediaSource API limitations with MP3 streaming)
  - ✅ Safari (may work)
  - Recommendation: Use Chrome-based browsers for web interface

### API Testing
```bash
# Direct API call (works with any HTTP client)
curl -X POST http://127.0.0.1:8880/v1/audio/speech \
  -H "Content-Type: application/json" \
  -d '{"input": "Hello from Kokoro!", "voice": "af_nova"}' \
  --output test.mp3

# OpenAI-compatible format
curl -X POST http://127.0.0.1:8880/v1/audio/speech \
  -H "Content-Type: application/json" \
  -d '{
    "model": "tts-1",
    "input": "Hello from Kokoro!",
    "voice": "nova"
  }' \
  --output test.mp3
```