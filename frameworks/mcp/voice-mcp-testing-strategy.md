---
source: original testing documentation
project: https://github.com/livekit/voice-mcp
related: https://github.com/modelcontextprotocol/fastmcp
---

# Voice-MCP Testing Strategy

## Overview

This document outlines the automated testing approach for voice-mcp, an MCP server that provides voice interaction capabilities.

## Testing Framework

We use **FastMCP's in-memory testing** approach combined with **pytest** for comprehensive test coverage.

### Key Technologies

- **pytest**: Main testing framework
- **pytest-asyncio**: For async test support
- **pytest-mock**: For mocking external dependencies
- **FastMCP Client**: For testing MCP protocol interactions

## Test Structure

```
python-package/
├── tests/
│   ├── __init__.py
│   ├── test_voice_mcp.py    # Main test suite
│   └── fixtures/             # Test fixtures (if needed)
├── pytest.ini                # Pytest configuration
└── Makefile                  # Test automation commands
```

## Test Categories

### 1. Unit Tests

Test individual components in isolation:

- **Tool functionality**: Each MCP tool (`speak_text`, `listen_for_speech`, etc.)
- **Audio processing**: Data type conversions, format handling
- **Configuration**: Environment variable loading, debug mode

### 2. Integration Tests

Test complete workflows:

- **Full conversation flow**: Question → Response → Confirmation
- **Transport selection**: Auto-detection and fallback logic
- **Error propagation**: API failures → User-friendly messages

### 3. Mock Strategy

Since audio I/O requires hardware, we mock:

- **OpenAI API calls**: STT and TTS responses
- **Audio hardware**: Recording and playback functions
- **LiveKit connections**: Room availability checks

## Key Test Patterns

### 1. MCP Server Testing Pattern

```python
@pytest.fixture
async def voice_mcp_server():
    """Create server with mocked dependencies"""
    with patch('get_openai_clients', return_value=mock_clients):
        return configured_mcp_server

async def test_tool(voice_mcp_server):
    async with Client(voice_mcp_server) as client:
        result = await client.call_tool("tool_name", params)
        assert expected in result[0].text
```

### 2. Audio Mocking Pattern

```python
@pytest.fixture
def mock_audio_functions():
    with patch('sounddevice.rec') as mock_rec:
        mock_rec.return_value = fake_audio_data
        yield mock_rec
```

### 3. Error Testing Pattern

```python
async def test_api_error(mock_openai_clients):
    mock_openai_clients['tts'].create.side_effect = Exception("API Error")
    result = await client.call_tool("speak_text", {"text": "Test"})
    assert "Error" in result[0].text
```

## Running Tests

### Local Development

```bash
# Install test dependencies
pip install -e ".[test]"

# Run all tests
pytest

# Run with coverage
pytest --cov=src/voice_mcp --cov-report=html

# Run specific test
pytest tests/test_voice_mcp.py::TestVoiceMCPTools::test_speak_text
```

### Continuous Integration

Tests run automatically on:
- Push to master/main/test branches
- Pull requests
- Multiple Python versions (3.8-3.12)
- Multiple OS (Ubuntu, macOS)

## Test Coverage Goals

- **Line coverage**: >80%
- **Branch coverage**: >70%
- **Critical paths**: 100% (audio playback, error handling)

## Future Improvements

1. **Performance tests**: Measure latency and resource usage
2. **Stress tests**: Multiple concurrent clients
3. **Hardware tests**: Optional tests with real audio devices
4. **End-to-end tests**: Full integration with Claude Desktop

## Best Practices

1. **Fast execution**: Use mocks to avoid slow I/O operations
2. **Deterministic**: No flaky tests dependent on timing
3. **Clear assertions**: Explicit error messages
4. **Isolated tests**: No shared state between tests
5. **CI-friendly**: No GUI or hardware requirements