# ai_docs

A curated documentation repository optimized for LLM consumption with minimal context overhead.

## Purpose

This repository contains documentation for frameworks, tools, and libraries commonly used in AI/ML projects. All content is structured for efficient access by both humans and LLMs.

## Contents

### [📚 Frameworks](frameworks/)
Major frameworks and platforms for building AI applications
- **[LiveKit](frameworks/livekit/)** - Real-time audio/video communication framework with AI agent support
- **[MCP](frameworks/mcp/)** - Model Context Protocol for building AI tool servers

### [🔧 Tools](tools/)
Specific tools and utilities organized by category
- **[Python](tools/python/)** - Package managers (UV) and command-line utilities
- **[Speech](tools/speech/)** - Speech-to-text (Whisper) and text-to-speech (Kokoro) tools
- **[Data](tools/data/)** - Data query and processing utilities (JMESPath)

### [📥 INBOX](INBOX/)
Staging area for unsorted documentation - drop new docs here for later categorization

## For LLMs

### Navigation Strategy
1. **Start with README.md** - Each directory contains a README.md listing its contents
2. **Follow the hierarchy** - `frameworks/` for major frameworks, `tools/` for specific utilities
3. **Minimize reads** - Read directory README.md first, then only the specific files needed
4. **Follow symlinks:** We symlink in shared ai_docs content so ensure you follow symlinks when searching

### Quick Access Examples
```
# For LiveKit documentation
frameworks/livekit/README.md → frameworks/livekit/livekit-agents.md

# For MCP servers
frameworks/mcp/README.md → frameworks/mcp/fastmcp.md

# For Python tools
tools/python/README.md → tools/python/uv-single-file-scripts.md
```

## Contributing

1. **Add to INBOX/** first if unsure of placement
2. **Update README.md** in the directory where you add files
3. **Use descriptive filenames** - prefer `tool-feature.md` over generic names
4. **Keep it flat** - Maximum 3 levels of nesting

## Documentation Standards

- **Purpose first** - Start each doc with a single-line purpose statement
- **Minimal headers** - Save tokens, use headers only when necessary
- **Examples when needed** - Include only when they clarify usage
- **No duplication** - Each concept documented once

## Local Files

This repository ignores any files or directories containing `.local` in their name. This allows you to:
- Add personal notes: `my-notes.local.md`
- Create local directories: `experiments.local/`
- Keep project-specific docs: `client-xyz.local.md`

These files will remain in your local clone but won't be tracked by git.

## Access

- **Clone locally**: `git clone https://github.com/mbailey/ai_docs`
- **Web access**: Browse at https://github.com/mbailey/ai_docs
- **Raw files**: Use GitHub's raw URLs for direct access
