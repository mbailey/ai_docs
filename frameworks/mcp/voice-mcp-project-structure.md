---
source: original project documentation
project: https://github.com/livekit/voice-mcp
---

# Voice-MCP Project Structure

## Script Organization

The voice-mcp UV script has a single source of truth to avoid duplication and sync issues:

- **Source**: `python-package/src/voice_mcp/scripts/voice-mcp` (authoritative version)
- **Symlink**: `mcp-servers/voice-mcp` → points to the source

## Why This Structure?

1. **Development**: The `.mcp.json` file references `./mcp-servers/voice-mcp` for local development
2. **Distribution**: The Python package includes the script in the correct location
3. **Maintenance**: Only one file to update when making changes

## Usage Patterns

### Local Development (cloned repo)
```bash
# MCP tools use the symlink
claude  # Uses .mcp.json → ./mcp-servers/voice-mcp
```

### Installed Package
```bash
# Installed via pip/uvx
voice-mcp  # Uses python package → runs uv with packaged script
```

### Direct UV Execution
```bash
# Run directly with uv
uv run --script mcp-servers/voice-mcp
```

All three methods use the same underlying script, ensuring consistency.