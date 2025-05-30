# Whisper Local Configuration

## Using Local Whisper with OpenAI Client

When whisper.cpp server is running locally, you can use it as a drop-in replacement for OpenAI's Whisper API by simply changing the `base_url` parameter:

### Python Example

```python
from openai import OpenAI

# For OpenAI's Whisper API (default)
client = OpenAI()

# For local whisper.cpp server
client = OpenAI(
    api_key="not-needed",  # Local server doesn't need API key
    base_url="http://localhost:8000/v1"
)

# Usage is identical
audio_file = open("audio.wav", "rb")
transcription = client.audio.transcriptions.create(
    model="whisper-1",  # Model is ignored, uses server's loaded model
    file=audio_file
)
print(transcription.text)
```

### In LiveKit Agents

```python
from livekit.plugins import openai

# Local whisper
stt = openai.STT(
    base_url="http://localhost:8000/v1",
    model="whisper-1"
)

# Or with environment variable
# OPENAI_BASE_URL=http://localhost:8000/v1
stt = openai.STT(model="whisper-1")
```

### Environment Variables

You can set the base URL via environment variables:

```bash
# .env.local
OPENAI_BASE_URL=http://localhost:8000/v1
```

### Automatic Fallback Example

```python
import os
import requests
from livekit.plugins import openai

def get_stt_provider():
    """Get STT provider with automatic fallback to OpenAI"""
    whisper_url = os.getenv("WHISPER_BASE_URL", "http://localhost:8000")
    
    # Check if local whisper is available
    try:
        response = requests.get(f"{whisper_url}/health", timeout=1)
        if response.status_code == 200:
            return openai.STT(base_url=f"{whisper_url}/v1", model="whisper-1")
    except:
        pass
    
    # Fallback to OpenAI
    return openai.STT(model="whisper-1")
```

## Starting Whisper Server

```bash
# Start whisper server (runs on port 8000 by default)
make whisper-start

# Or manually
./libexec/whisper-start

# Check if it's running
curl http://localhost:8000/health

# Stop when done
make whisper-stop
```

## Configuration Options

### Port Configuration
```bash
# Use different port
WHISPER_PORT=8001 ./libexec/whisper-start
```

### Model Selection
The model is determined by hardware detection at build time:
- CPU: `large-v3-turbo`
- GPU: `large-v3`
- Apple Silicon: `large-v3-mlx`

## Performance Comparison

| Configuration | Latency | Cost |
|--------------|---------|------|
| OpenAI API | ~500ms | $0.006/min |
| Local CPU | ~1000ms | Free |
| Local GPU | ~200ms | Free |
| Local Apple Silicon | ~300ms | Free |

## Complete Example

```python
#!/usr/bin/env python3
import os
from livekit.agents import Agent
from livekit.plugins import openai, anthropic

# Automatically use local whisper if available
whisper_base = os.getenv("WHISPER_BASE_URL", "http://localhost:8000/v1")
kokoro_base = os.getenv("KOKORO_BASE_URL", "http://localhost:8880/v1")

agent = Agent(
    stt=openai.STT(base_url=whisper_base, model="whisper-1"),
    llm=anthropic.LLM(),
    tts=openai.TTS(base_url=kokoro_base, model="kokoro", voice="af_sky")
)
```

## Notes

- The local whisper server ignores the `model` parameter and uses the model loaded at startup
- No API key is required for local servers
- Audio format conversion is handled automatically by the server
- The server supports the same audio formats as OpenAI (wav, mp3, mp4, m4a, flac, ogg, webm)