# LiveKit Agents API: Comprehensive Guide for LLMs

This document provides deep technical understanding of LiveKit Agents API for building voice and multimodal AI applications.

## Architecture Overview

### Core Concepts

**Worker-Job-Agent Model:**
- **Worker**: Main process that registers with LiveKit server, handles multiple jobs
- **Job**: Individual session process spawned for each room/user interaction  
- **Agent**: The AI participant that joins the room and handles voice/multimodal interactions

**Key Insight**: LiveKit Agents uses a distributed worker model where agents are deployed automatically by the server, not manually connected to specific rooms.

### Two Deployment Patterns

#### 1. Worker Pattern (Production/Standard)
```python
# Standard worker deployment - agents get assigned to rooms automatically
def prewarm(proc: JobProcess):
    proc.userdata["vad"] = silero.VAD.load()

async def entrypoint(ctx: JobContext):
    await ctx.connect(auto_subscribe=AutoSubscribe.AUDIO_ONLY)
    participant = await ctx.wait_for_participant()
    
    session = AgentSession(vad=ctx.proc.userdata["vad"])
    await session.start(room=ctx.room, agent=Assistant())

cli.run_app(WorkerOptions(entrypoint_fnc=entrypoint, prewarm_fnc=prewarm))
```

#### 2. Direct Room Connection (MCP/Custom Use Cases)
```python
# Direct connection to specific room - manual room targeting
room = rtc.Room()
await room.connect(LIVEKIT_URL, token.to_jwt())

session = AgentSession(vad=silero.VAD.load())
await session.start(room=room, agent=Agent())
```

## Agent Classes and Patterns

### 1. Standard Agent (Voice Pipeline)
```python
class Assistant(Agent):
    def __init__(self):
        super().__init__(
            instructions="Your system prompt",
            stt=openai.STT(model="whisper-1"),           # Speech-to-Text
            llm=openai.LLM(model="gpt-4o-mini"),         # Language Model
            tts=openai.TTS(model="tts-1", voice="nova"), # Text-to-Speech
            turn_detection=MultilingualModel(),           # Turn detection
        )
    
    async def on_enter(self):
        """Called when agent enters room"""
        self.session.generate_reply("Hello! How can I help?", allow_interruptions=True)
```

### 2. MultimodalAgent (Realtime Model)
```python
def run_multimodal_agent(ctx: JobContext, participant: rtc.RemoteParticipant):
    model = openai.realtime.RealtimeModel(
        instructions="Your system prompt",
        modalities=["audio", "text"],
    )
    
    agent = MultimodalAgent(model=model, chat_ctx=llm.ChatContext())
    agent.start(ctx.room, participant)
    agent.generate_reply()
```

## Critical Components

### STT (Speech-to-Text) Providers and Patterns

**Available STT Providers**:
- Amazon Transcribe, AssemblyAI, Azure AI Speech, Azure OpenAI
- Clova, Deepgram, fal, Gladia, Google Cloud, Groq  
- **OpenAI** (Whisper models), Speechmatics

**Streaming vs Non-Streaming**:
```python
# Streaming STT (real-time)
stt = deepgram.STT(model="nova-2")

# Non-streaming STT (requires VAD)
stt = openai.STT(model="whisper-1")  # Needs VAD for streaming
```

**STT Stream Implementation**:
```python
stt_stream = stt.stream()
audio_stream = rtc.AudioStream(track)

async for audio_event in audio_stream:
    stt_stream.push_frame(audio_event.frame)
    # Generates SpeechEvent outputs (interim/final transcripts)
```

### TTS (Text-to-Speech) Providers and Patterns

**Available TTS Providers**:
- Amazon Polly, Azure AI Speech, Azure OpenAI, Cartesia
- Deepgram, ElevenLabs, Google Cloud, Groq, Hume
- Neuphonic, **OpenAI**, PlayHT, Resemble AI, Rime, Speechify

**TTS Stream Implementation**:
```python
tts = cartesia.TTS(model="sonic-english")
tts_stream = tts.stream()
tts_stream.push_text(text)
tts_stream.end_input()

# Consume audio frames and publish to room
```

**Voice Configuration**:
```python
# OpenAI TTS with voice selection
tts = openai.TTS(model="tts-1", voice="nova")

# ElevenLabs with custom voice
tts = elevenlabs.TTS(voice="rachel")
```

### Voice Activity Detection (VAD)
**Essential for non-streaming STT**:
```python
# Always include VAD for non-realtime STT (like OpenAI Whisper)
vad = silero.VAD.load()
session = AgentSession(vad=vad)

# Without VAD, you'll get:
# RuntimeError: STT does not support streaming, add VAD to enable streaming

# VAD uses StreamAdapter to buffer speech segments for non-streaming models
```

### AgentSession Configuration
```python
session = AgentSession(
    vad=vad,                          # Required for streaming STT
    min_endpointing_delay=0.5,        # Min delay when turn detector thinks user is done
    max_endpointing_delay=5.0,        # Max delay when turn detector unsure
)
```

### Room Input Options
```python
room_input_options = RoomInputOptions(
    noise_cancellation=noise_cancellation.BVC(),  # Background voice cancellation
)
```

## Pipeline Architecture

### STT � LLM � TTS Flow
1. **Audio Input** � VAD detects speech
2. **STT** converts speech to text chunks
3. **LLM** processes text and generates response
4. **TTS** converts response to audio
5. **Audio Output** � User hears response

### Turn Detection
- **Purpose**: Determines when user has finished speaking
- **Types**: 
  - `MultilingualModel()` - Transformer-based, more accurate
  - `EOUModel()` - End-of-utterance detection
- **Impact**: Affects responsiveness and interruption handling

## Event Lifecycle

### Agent Lifecycle Methods
```python
class CustomAgent(Agent):
    async def on_enter(self):
        """Called when agent joins room"""
        pass
    
    async def on_chat_message(self, message: str):
        """Called when receiving text/transcribed voice"""
        pass
    
    async def on_participant_connected(self, participant):
        """Called when new participant joins"""
        pass
```

### Session Events
```python
session.on("metrics_collected", on_metrics_collected)
session.on("participant_connected", on_participant_connected)
```

## Common Patterns and Anti-Patterns

###  Correct Patterns

#### Worker Deployment
```python
# Let LiveKit manage room assignment
async def entrypoint(ctx: JobContext):
    await ctx.connect(auto_subscribe=AutoSubscribe.AUDIO_ONLY)
    participant = await ctx.wait_for_participant()
    # Agent automatically deployed to any room when user joins
```

#### VAD Integration
```python
# Always include VAD for streaming STT
def prewarm(proc: JobProcess):
    proc.userdata["vad"] = silero.VAD.load()

session = AgentSession(vad=ctx.proc.userdata["vad"])
```

#### Proper Agent Initialization
```python
class Assistant(Agent):
    def __init__(self):
        super().__init__(
            instructions="Clear, specific instructions",
            stt=provider.STT(),
            llm=provider.LLM(),
            tts=provider.TTS(),
        )
```

### L Anti-Patterns

#### Manual Audio Handling
```python
# Don't do this - let Agent framework handle audio
tts_result = await tts.synthesize(text)
source = rtc.AudioSource(tts_result.sample_rate, tts_result.num_channels)
# This breaks the pipeline and causes streaming errors
```

#### Missing VAD
```python
# Don't do this - will cause streaming errors
session = AgentSession()  # Missing VAD!
# Results in: RuntimeError: STT does not support streaming
```

#### Hardcoded Room Names in Workers
```python
# Don't do this in worker mode
ROOM_NAME = "specific-room"  # Workers should be flexible
```

## MCP Integration Patterns

### Challenge: Worker vs Direct Connection
**Problem**: MCP servers need to target specific rooms, but worker pattern auto-assigns rooms.

**Solution**: Use direct room connection pattern for MCP:

```python
async def mcp_voice_interaction(question: str, room_name: str = None):
    # Auto-discover active room if not specified
    if not room_name:
        room_name = await auto_discover_active_room()
    
    # Create token for specific room
    token = api.AccessToken(API_KEY, API_SECRET)
    token.with_identity("mcp-bot")
    token.with_grants(api.VideoGrants(room_join=True, room=room_name))
    
    # Direct room connection
    room = rtc.Room()
    await room.connect(LIVEKIT_URL, token.to_jwt())
    
    # Use Agent framework for audio pipeline
    vad = silero.VAD.load()
    session = AgentSession(vad=vad)
    await session.start(room=room, agent=QuestionAgent(question))
```

### Room Discovery Pattern
```python
async def auto_discover_active_room() -> str:
    """Find room with active participants for MCP targeting"""
    lk_api = api.LiveKitAPI(API_URL, API_KEY, API_SECRET)
    rooms = await lk_api.room.list_rooms(api.ListRoomsRequest())
    
    for room in rooms.rooms:
        if room.num_participants > 0:
            return room.name
    return ""
```

## Response Collection Patterns

### Issue: Capturing Agent Responses
**Problem**: How to get text back from voice responses in MCP context.

**Current Solutions**:
1. **Event-based**: Listen for `on_chat_message` events
2. **Chat context**: Access agent's chat history
3. **Custom callbacks**: Implement response collection mechanisms

```python
class ResponseCollector(Agent):
    def __init__(self, question: str, response_callback):
        self.response_callback = response_callback
        super().__init__(instructions=f"Answer this question: {question}")
    
    async def on_chat_message(self, message: str):
        """Capture responses and send to callback"""
        await self.response_callback(message)
```

## Debugging Common Issues

### STT Streaming Errors
```
ERROR: STT does not support streaming, add VAD to enable streaming
```
**Fix**: Always include VAD in AgentSession:
```python
vad = silero.VAD.load()
session = AgentSession(vad=vad)
```

### Audio Pipeline Breaks
```
ERROR: object ChunkedStream can't be used in 'await' expression
```
**Fix**: Don't manually handle TTS audio - let Agent framework manage it:
```python
# Wrong:
tts_result = await self.tts.synthesize(text)
# Right:
self.session.generate_reply(instructions=text)
```

### Room Connection Issues
**Symptoms**: Agent connects but can't hear/speak
**Fixes**:
1. Ensure correct room name
2. Check token permissions
3. Verify audio subscription settings
4. Confirm VAD is loaded

## Best Practices

### 1. Resource Management
- Use `prewarm_fnc` for model loading
- Implement proper cleanup in job shutdown
- Monitor memory usage with multiple concurrent sessions

### 2. Error Handling
```python
async def entrypoint(ctx: JobContext):
    try:
        await ctx.connect(auto_subscribe=AutoSubscribe.AUDIO_ONLY)
        # ... agent logic
    except Exception as e:
        logger.error(f"Agent error: {e}")
        # Graceful cleanup
```

### 3. Configuration Management
```python
# Use environment variables for flexibility
STT_PROVIDER = os.getenv("STT_PROVIDER", "openai")
LLM_MODEL = os.getenv("LLM_MODEL", "gpt-4o-mini")
TTS_VOICE = os.getenv("TTS_VOICE", "nova")
```

### 4. Monitoring and Metrics
```python
def on_metrics_collected(agent_metrics: metrics.AgentMetrics):
    metrics.log_metrics(agent_metrics)
    # Custom monitoring logic
```

## Architecture Decision Guide

### When to Use Each Pattern

| Use Case | Pattern | Key Features |
|----------|---------|--------------|
| Production Voice Assistant | Worker + Agent | Auto-scaling, load balancing, room auto-assignment |
| Realtime Multimodal | Worker + MultimodalAgent | Single model handles STT+LLM+TTS |
| MCP Integration | Direct Connection + Agent | Specific room targeting, custom response collection |
| Custom Voice Bot | Direct Connection + Custom Agent | Full control over room selection and behavior |

### Performance Considerations
- **Worker Pattern**: Better for high-scale production (multiple concurrent sessions)
- **Direct Connection**: Better for targeted interactions (MCP, testing, demos)
- **VAD Impact**: Adds ~100ms latency but enables streaming STT
- **Model Choice**: Realtime models reduce latency but limit provider flexibility

## Fixing bin/livekit-voice-mcp-simple

### Current Issues Analysis

**Problem 1: Response Collection Anti-Pattern**
```python
# Current broken approach in our MCP:
async def on_chat_message(self, message: str):
    responses[self.response_id] = message  # Global dict anti-pattern
    pending_responses[self.response_id].set()  # Event-based collection
```

**Problem 2: Dual Agent Confusion**
- Our QuestionAgent speaks the question via TTS
- User's main livekit-agent hears and responds
- Our QuestionAgent tries to capture the response
- **Result**: Two agents in conversation, not agent-to-human

**Problem 3: Pipeline Interference**
```python
# Our agent has its own LLM pipeline:
llm=openai.LLM(model="gpt-4o-mini"),  # This processes the question!

# When it should just speak and listen:
# No LLM needed for a question-only bot
```

### Correct MCP Voice Pattern

#### Option 1: Speaker-Only Bot (Recommended)
```python
class SpeakerBot(Agent):
    """Bot that only speaks - no LLM processing"""
    def __init__(self, message: str, response_callback=None):
        self.message = message
        self.response_callback = response_callback
        super().__init__(
            instructions="You are a message delivery bot",
            stt=openai.STT(model="whisper-1"),  # For listening only
            llm=None,  # No LLM - don't process responses
            tts=openai.TTS(model="tts-1", voice="nova"),
        )
    
    async def on_enter(self):
        # Speak message directly via TTS, no LLM processing
        # Use the raw TTS stream for direct control
        tts_stream = self.tts.stream()
        tts_stream.push_text(self.message)
        tts_stream.end_input()
        
    async def on_chat_message(self, message: str):
        # Capture human responses and send to callback
        if self.response_callback:
            await self.response_callback(message)
        logger.info(f"Human said: {message}")
```

#### Option 2: Manual TTS/STT Integration
```python
class ManualVoiceBot:
    """Direct TTS/STT control without Agent pipeline"""
    def __init__(self, room: rtc.Room):
        self.room = room
        self.tts = openai.TTS(model="tts-1", voice="nova")
        self.stt = openai.STT(model="whisper-1")
        self.vad = silero.VAD.load()
    
    async def speak(self, text: str):
        """Speak text directly to room"""
        tts_stream = self.tts.stream()
        tts_stream.push_text(text)
        tts_stream.end_input()
        
        # Publish audio frames to room
        async for audio_frame in tts_stream:
            # Custom audio track publishing logic
            pass
    
    async def listen(self) -> str:
        """Listen for voice response"""
        # Set up STT stream with VAD
        # Process audio from room participants
        # Return transcribed text
        pass
```

### Alternative: Data Message Pattern

Instead of voice, use LiveKit's data messages for MCP:

```python
@mcp.tool()
async def send_message_to_room(message: str, room_name: str = "") -> str:
    """Send text message to room participants"""
    if not room_name:
        room_name = await auto_discover_active_room()
    
    lk_api = api.LiveKitAPI(API_URL, API_KEY, API_SECRET)
    
    await lk_api.room.send_data(
        api.SendDataRequest(
            room=room_name,
            data=json.dumps({
                "type": "mcp_message", 
                "content": message,
                "sender": "claude-mcp"
            }).encode('utf-8'),
            kind=api.DataPacketKind.KIND_RELIABLE,
        )
    )
    
    return f"Message sent to room {room_name}"
```

This guide provides the foundation for understanding and debugging LiveKit Agents implementations, especially in complex scenarios like MCP integration where standard patterns need adaptation.