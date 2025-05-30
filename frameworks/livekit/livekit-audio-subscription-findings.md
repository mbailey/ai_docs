# LiveKit Audio Subscription Deep Dive Findings

## Critical Difference Between Working Agent and MCP

### Working Agent (bin/livekit-agent)
1. Uses `JobContext` pattern with worker architecture
2. **Key step**: `await ctx.wait_for_participant()` - waits for a human participant to join BEFORE starting the agent
3. Uses `auto_subscribe=AutoSubscribe.AUDIO_ONLY` to only subscribe to audio tracks
4. AgentSession automatically creates RoomIO when given a room

### MCP Implementation Issue
1. Bot joins room immediately without waiting for participants
2. Bot speaks its message as the ONLY participant in the room
3. RoomIO's audio input stream needs a participant to subscribe to

## How Audio Subscription Works

### RoomIO Initialization (_room_io.py)
1. `_ParticipantAudioInputStream` is created with `track_source=SOURCE_MICROPHONE`
2. It registers handlers for `track_subscribed` and `track_unpublished` events
3. **Critical**: It needs `set_participant()` to be called with a participant identity

### Participant Tracking Flow
1. RoomIO waits for participants via `_participant_available_fut`
2. When a participant connects, `_on_participant_connected` checks:
   - If specific identity matches (if set)
   - If participant is acceptable type (SIP, STANDARD)
   - Skips participants publishing on behalf of the agent
3. Once participant is available, `set_participant()` is called
4. This triggers audio input to subscribe to that participant's microphone track

### The Missing Piece
The MCP bot doesn't wait for or track human participants properly. When it speaks, it's alone in the room. Even if a human is present, the bot hasn't:
1. Waited for them to be ready
2. Called `set_participant()` to subscribe to their audio
3. Set up the proper audio subscription flow

## Agent on_chat_message Trigger
- STT processes audio from subscribed tracks
- When speech is detected and transcribed, it triggers `on_chat_message`
- But this ONLY works if:
  1. A participant's audio track is subscribed
  2. The audio input stream is properly forwarding frames
  3. The VAD/STT pipeline is processing the audio

## Solution for MCP
The MCP needs to either:
1. Wait for a participant before speaking (like the working agent)
2. Manually handle participant tracking and audio subscription
3. Use a different pattern that doesn't rely on RoomIO's automatic participant handling

The current implementation speaks into an empty room or doesn't properly subscribe to participant audio tracks, so STT never receives audio to process.