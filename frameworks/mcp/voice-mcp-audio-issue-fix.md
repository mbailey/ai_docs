# Voice-MCP Audio Issue Fix

## Problem Description

When running voice-mcp through Claude Code without the `--debug` and `--debug-mcp` flags, audio playback does not work. The server runs but no sound is produced when using the TTS functionality.

## Root Cause

The issue is caused by the `sounddevice` library's internal stderr redirection. The library includes a function `_ignore_stderr()` that redirects stderr to `/dev/null` to suppress PortAudio error messages. This redirection interferes with audio playback when running in an MCP server context where stdout/stderr are captured or redirected.

### Technical Details

1. **sounddevice's stderr redirection**: The library redirects stderr to suppress PortAudio messages
2. **MCP server context**: When running without debug flags, the MCP server may buffer or redirect stdout/stderr
3. **Conflict**: The combination causes audio playback to fail silently

## Solution Implemented

### 1. Disable sounddevice stderr redirection

```python
# Workaround for sounddevice stderr redirection issue
if hasattr(sd._sounddevice, '_ignore_stderr'):
    sd._sounddevice._ignore_stderr = lambda: None
```

This prevents sounddevice from redirecting stderr, allowing audio to work properly in all contexts.

### 2. Improved error handling and fallback methods

Added multiple fallback audio playback methods:
- Primary: sounddevice with proper initialization
- Secondary: PyDub playback (using simpleaudio)
- Tertiary: Save audio file for manual playback

### 3. Better audio device initialization

```python
# Force initialization before playing
sd.default.samplerate = audio.frame_rate
sd.default.channels = audio.channels
```

## Testing the Fix

1. **Without debug flags**: `claude code` - Audio should now work
2. **With debug flags**: `claude code --debug --debug-mcp` - Audio continues to work
3. **Error scenarios**: Graceful fallback to alternative methods

## References

- [sounddevice issue #166](https://github.com/spatialaudio/python-sounddevice/issues/166)
- [sounddevice issue #461](https://github.com/spatialaudio/python-sounddevice/issues/461)