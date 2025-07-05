---
source: https://spec.modelcontextprotocol.io/specification/architecture/#transports
---

# MCP Specification - Transports and Lifecycle

## Transports

MCP defines two primary transport mechanisms:

### stdio Transport

When using stdio:
- Client launches the server as a subprocess
- Messages are exchanged via standard input/output streams
- All messages MUST be:
  - JSON-RPC 2.0 encoded
  - UTF-8 encoded text
  - Delimited by newlines
  - Free of embedded newlines

Key requirements:
- Server reads from `stdin` and writes to `stdout`
- Server MAY write logging/debug info to `stderr`
- Server MUST NOT write non-MCP messages to `stdout`
- Client MUST NOT write non-MCP messages to server's `stdin`

### Connection Lifecycle

The stdio connection has three phases:

1. **Initialization**: Client and server negotiate protocol version and capabilities
2. **Operation**: Normal protocol communication - **THE CONNECTION REMAINS OPEN**
3. **Shutdown**: Graceful termination

### Shutdown Process

For stdio transport, shutdown occurs when:

**Client-initiated shutdown:**
1. Client closes the input stream to the server
2. Client waits for server to exit
3. If server doesn't exit, client sends SIGTERM
4. As last resort, client sends SIGKILL

**Server-initiated shutdown:**
- Server MAY close its output stream and exit
- No specific shutdown messages are defined
- The transport mechanism signals termination

## Key Point: Persistent Connections

**The stdio connection should remain open during the entire session and handle multiple requests.** It should NOT disconnect after each request. This is explicit in the specification - the connection stays open during the "Operation" phase until explicitly shutdown.