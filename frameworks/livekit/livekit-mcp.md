# livekit mcp

Search through the external/livekit* for any files relevant to mcp:
- filenames containing mcp
- mcp in their body

Add notes below referencing libraries or examples that may be useful for this project.

---

## Search Results

**No MCP (Model Context Protocol) references found in LiveKit core repository.**

All instances of "mcp" in the codebase are part of CPU-related variable names (numCPUs, NumCpus, etc.).

## MCP Integration Opportunities

While LiveKit doesn't have built-in MCP support, these components could be useful for building MCP-enabled voice assistants:

1. **LiveKit Agents Framework** (`external/agents`)
   - Python and Node.js frameworks for building AI agents
   - Could be extended to communicate with MCP servers
   - Handles real-time voice/video streams

2. **Voice Assistant Examples** (from repos.txt)
   - `livekit-examples/voice-assistant-frontend` - Frontend template
   - `livekit-examples/multimodal-agent-python` - Multimodal capabilities
   - `livekit-examples/basic-mcp` - Potential MCP integration example

3. **Integration Architecture**
   - LiveKit handles real-time voice/video communication
   - MCP servers provide tool capabilities (file access, web search, etc.)
   - Bridge component needed to connect LiveKit agents with MCP servers
