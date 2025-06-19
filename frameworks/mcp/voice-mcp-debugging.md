# Voice MCP Debugging Guide

## BrokenResourceError Investigation

### Error Details
```
anyio.BrokenResourceError
File: mcp/server/stdio.py, line 71, in stdin_reader
```

This error occurs when the stdin/stdout pipe between Claude CLI and the MCP server is broken.

### Common Causes

1. **Audio Library Interference**
   - `sounddevice` library may redirect stderr
   - Audio operations can interfere with stdio streams
   - File descriptors may be corrupted during audio I/O

2. **Concurrent Operations**
   - Multiple tool calls arriving simultaneously
   - Audio recording/playback operations overlapping
   - No locking mechanism for audio resources

3. **Timing Issues**
   - New tool call arrives while previous audio is processing
   - Race conditions between audio completion and new requests

### Debugging Steps

1. **Enable Trace Logging**
   ```bash
   export VOICE_MCP_DEBUG=trace
   mcp
   ```

2. **Check MCP Logs**
   ```bash
   ls -la ~/.cache/claude-cli-nodejs/-home-m-Code-github-com-mbailey-voice-mcp/mcp-logs-voice-mcp/
   ```

3. **Monitor stdio Health**
   - Check if stderr is being redirected
   - Verify file descriptors 0, 1, 2 remain intact
   - Look for any library warnings about stdio

### Potential Fixes

1. **Serialize Audio Operations**
   ```python
   audio_lock = asyncio.Lock()
   
   async def converse(...):
       async with audio_lock:
           # Perform audio operations
   ```

2. **Protect stdio Streams**
   ```python
   # Save original file descriptors
   orig_stderr = os.dup(2)
   
   # Restore after audio operation
   os.dup2(orig_stderr, 2)
   ```

3. **Enhanced Error Handling**
   ```python
   try:
       # MCP operations
   except anyio.BrokenResourceError:
       logger.error("Stdio pipe broken, attempting recovery")
       # Recovery logic
   ```

### Testing Scenarios

1. **Rapid Sequential Calls**
   - Call `converse` multiple times quickly
   - Test with varying audio durations

2. **Concurrent Calls**
   - Multiple `converse` calls simultaneously
   - Mix of `converse` and `listen_for_speech`

3. **Long-Running Operations**
   - Extended recording sessions (30-60 seconds)
   - Large TTS generations

### Known Issues

- `sounddevice` library interferes with stderr in some versions
- Python's asyncio may have issues with stdio in subprocesses
- MCP stdio transport requires persistent connection

### Current Protection Mechanisms

Based on code analysis, voice-mcp already has:

1. **Audio Operation Lock** (line 140)
   - `asyncio.Lock()` to serialize audio operations
   - Used in `converse()` and other audio functions

2. **Stdio Protection** (lines 270-320 in `record_audio`)
   - Saves original stdin/stdout/stderr
   - Restores them in finally block

3. **Sounddevice Workaround** (lines 38-74)
   - Comprehensive disable of stderr redirection
   - Multiple methods to prevent interference

4. **BrokenPipeError Handling** (line 734)
   - Catches BrokenPipeError in main()
   - But NOT catching `anyio.BrokenResourceError`

### Root Cause Analysis

The error is `anyio.BrokenResourceError` not `BrokenPipeError`:
- Occurs in MCP's stdio transport layer
- Happens when stdin_reader tries to send a session message
- Current error handling doesn't catch this specific exception

### Recommended Fix

Add `anyio.BrokenResourceError` to the exception handling:

```python
try:
    mcp.run()
except (BrokenPipeError, anyio.BrokenResourceError) as e:
    logger.error(f"Connection lost to MCP client: {e}")
    sys.exit(1)
```

### References

- [MCP Transport Specification](mcp-specification-transports.md)
- [FastMCP Documentation](fastmcp/)
- [Audio Issue Fix](../../audio-issue-fix.md)