---
source: https://github.com/ggerganov/whisper.cpp
---

# Whisper.cpp Integration Documentation

## Overview

Whisper.cpp is a high-performance C++ port of OpenAI's Whisper automatic speech recognition (ASR) model. This integration provides local, offline speech-to-text capabilities for the voice-mcp project.

## Architecture

### Components

1. **whisper-server**: Native HTTP server providing OpenAI-compatible API
   - Listens on port 2022
   - Endpoint: `/v1/audio/transcriptions`
   - Accepts multipart/form-data with audio files

2. **Hardware Detection**: Automatic selection of optimal models
   - CPU: Uses `base` model (148MB, good balance)
   - GPU: Uses `large-v3` model (best accuracy)
   - Apple Silicon: Uses MLX-optimized models

3. **Native Binary Approach**: No containers required
   - Direct execution for minimal overhead
   - Fast startup times
   - Lower resource usage

## Configuration

### Environment Variables
```bash
WHISPER_PORT=2022           # Server port (default: 2022)
WHISPER_MODEL=base         # Override model selection
```

### Model Selection
Models are automatically selected based on hardware:
- `base`: 148MB, ~5x realtime on CPU
- `small`: 466MB, better accuracy
- `medium`: 1.5GB, good for GPU
- `large-v3`: 3.1GB, best accuracy
- `large-v3-turbo`: 1.6GB, optimized large model

## Integration Points

### Agent.py
```python
stt=openai.STT(base_url="http://localhost:2022/v1", model="whisper-1")
```

### LiveKit Voice MCP
The MCP server uses the same configuration to ensure all voice processing uses local whisper.

### Fallback Mechanism
If whisper server is unavailable, the system can fall back to OpenAI's cloud API if configured.

## Management Scripts

- **whisper-build**: Builds whisper.cpp binary from source
- **whisper-start**: Starts the whisper server
- **whisper-stop**: Stops the whisper server
- **whisper-detect-hardware**: Detects optimal hardware configuration

## Performance

- **Startup Time**: ~2-3 seconds
- **Processing Speed**: 5-10x realtime on modern CPUs
- **Memory Usage**: 200-500MB depending on model
- **Accuracy**: Comparable to OpenAI's cloud API

## Troubleshooting

### Server Won't Start
- Check if port 2022 is available
- Verify model file exists in `models/` directory
- Check logs at `/tmp/whisper-server.log`

### Poor Transcription Quality
- Try a larger model (small or medium)
- Ensure audio is 16kHz sample rate
- Check CPU usage - may need to reduce concurrent requests

### Hardware Detection Issues
- Manually specify model with `WHISPER_MODEL` env var
- Check `nvidia-smi` for GPU detection
- Verify Apple Silicon detection on macOS

## API Compatibility

The whisper-server provides an OpenAI-compatible API:

```bash
curl -X POST http://localhost:2022/v1/audio/transcriptions \
  -H "Content-Type: multipart/form-data" \
  -F "file=@audio.wav" \
  -F "model=whisper-1"
```

Response format matches OpenAI:
```json
{
  "text": "transcribed text here",
  "language": "en",
  "duration": 2.5
}
```