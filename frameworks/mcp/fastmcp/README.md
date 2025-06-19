# FastMCP Documentation

FastMCP is a Python library for rapidly building Model Context Protocol (MCP) servers. It provides a simple, decorator-based API for creating MCP-compliant servers that can expose tools, resources, and prompts to AI assistants.

## Key Features

- **Simple API**: Use decorators like `@mcp.tool()` to expose functions as MCP tools
- **Automatic JSON-RPC handling**: FastMCP manages all the protocol details
- **Built-in transport support**: Stdio and HTTP transports out of the box
- **Type safety**: Automatic validation and type conversion
- **Async support**: Full async/await support for all operations

## Quick Start

```python
from fastmcp import FastMCP

mcp = FastMCP("My Server")

@mcp.tool()
async def hello(name: str) -> str:
    """Say hello to someone"""
    return f"Hello, {name}!"

if __name__ == "__main__":
    mcp.run()
```

## Transport Behavior

### Stdio Transport
- Server runs as a subprocess of the client
- Communication via stdin/stdout
- **Connection persists for entire session**
- Server should NOT exit after handling requests
- Only terminates on explicit shutdown signal

### Common Issues

1. **Server disconnecting after requests**: This violates MCP spec. The stdio connection should remain open.
2. **BrokenResourceError**: Usually indicates the client closed the connection or the server exited unexpectedly
3. **Event loop errors on shutdown**: Can occur if cleanup handlers try to create new event loops

## Best Practices

1. Don't call `sys.exit()` or similar in tool handlers
2. Handle exceptions gracefully without terminating
3. Use proper async context management
4. Keep the main event loop running until shutdown

## LLM-Specific Documentation

For LLMs working with FastMCP code:
- **[llms.md](llms.md)** - Concise reference for common FastMCP patterns
- **[llms-full.md](llms-full.md)** - Comprehensive FastMCP documentation with examples

## References

- [MCP Specification](https://modelcontextprotocol.io/specification)
- [FastMCP GitHub](https://github.com/modelcontextprotocol/fastmcp)
- [Transport Specification](../mcp-specification-transports.md)
