# MCP (Model Context Protocol) Documentation

Build tool servers that AI assistants can interact with using the Model Context Protocol.

## Available Documentation

### Core MCP Documentation
- **[fastmcp/](fastmcp/README.md)** - FastMCP library documentation for rapidly building MCP servers
- **[interactive-mcp.md](interactive-mcp.md)** - Interactive MCP server patterns and examples
- **[mcp-specification-transports.md](mcp-specification-transports.md)** - MCP transport specification and lifecycle
- **[model-context-protocol-specification/](model-context-protocol-specification/)** - Full MCP specification documentation

### Voice MCP Documentation
- **[voice-mcp-debugging.md](voice-mcp-debugging.md)** - Debugging guide for voice-mcp implementations
- **[voice-mcp-audio-issue-fix.md](voice-mcp-audio-issue-fix.md)** - Fix for sounddevice audio issues in voice-mcp
- **[voice-mcp-project-structure.md](voice-mcp-project-structure.md)** - Project structure for voice-mcp server
- **[voice-mcp-testing-strategy.md](voice-mcp-testing-strategy.md)** - Testing approach for voice-mcp implementations

## Quick Reference

### MCP Fundamentals
- **Protocol**: JSON-RPC 2.0 based, stateful sessions
- **Transports**: Stdio (subprocess) and HTTP (network)
- **Components**: Tools, Resources, and Prompts

### Building MCP Servers
- Quick start with FastMCP: [fastmcp/README.md](fastmcp/README.md)
- Interactive patterns: [interactive-mcp.md](interactive-mcp.md)

### Key Concepts

#### Stdio Transport Behavior
- Server runs as subprocess of client
- **Connection persists throughout session** (does NOT disconnect after requests)
- Communication via stdin/stdout
- Only terminates on explicit shutdown

#### Common Issues
1. **Server disconnecting**: Violates MCP spec - stdio connections must persist
2. **BrokenResourceError**: Client closed connection or server exited unexpectedly
3. **Cleanup errors**: Event loop issues during shutdown

### Integration
- LiveKit + MCP: ../livekit/livekit-mcp-servers.md

## References
- [Official MCP Specification](https://modelcontextprotocol.io/specification)
- [MCP GitHub](https://github.com/modelcontextprotocol)