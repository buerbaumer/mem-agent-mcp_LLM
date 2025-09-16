# Copilot Instructions for Memory Agent MCP

## Project Overview

This is a **Model Context Protocol (MCP) server** that provides AI agents with an Obsidian-like memory system. The architecture has 3 main layers:
- **Agent Engine** (`agent/`): Core AI reasoning with sandboxed Python execution
- **MCP Server** (`mcp_server/`): Protocol implementation for external AI applications
- **Memory Connectors** (`memory_connectors/`): Extensible data ingestion from various sources

## Core Architecture Patterns

### Memory-First Agent Design
- The `Agent` class in `agent/agent.py` operates with a **structured response format**: `<think>`, `<python>`, `<reply>`
- All memory operations happen through Python code execution in a sandboxed environment (`agent/engine.py`)
- Memory is stored as **Markdown files with entity relationships**: `user.md` and `entities/[name].md`
- Memory paths are configured via `.memory_path` file (read by `_read_memory_path()` functions)

### Multi-Backend LLM Support
```python
# agent/model.py - Strategy pattern for different backends
if use_vllm:
    client = create_vllm_client(host=VLLM_HOST, port=VLLM_PORT)
else:
    client = create_openai_client()
```
- **macOS**: Uses MLX models (4-bit/8-bit quantization via `.mlx_model_name` file)
- **Linux**: Uses vLLM for local inference
- **Fallback**: OpenAI API via OpenRouter

### MCP Protocol Implementation
- `mcp_server/server.py`: FastMCP server for Claude Desktop/LM Studio integration
- `mcp_server/mcp_http_server.py`: HTTP/SSE server for ChatGPT integration
- Tools are registered as MCP functions that execute agent workflows

## Critical Development Workflows

### Setup & Configuration
```bash
make check-uv && make install    # Install with uv package manager
make setup                       # GUI to select memory directory → .memory_path
make run-agent                   # Start agent (prompts for MLX model on macOS)
make generate-mcp-json          # Generate MCP config for client apps
```

### Memory Management
- **Filters**: Use `make add-filters` to add privacy rules to `.filters` file
- **Memory structure**: Always use `[[entities/name.md]]` link format in user.md
- **Connectors**: Run `make memory-wizard` for interactive data source integration

### Testing & Debugging
- **CLI Testing**: `make chat-cli` for direct agent interaction
- **Server Testing**: `make serve-mcp` (FastMCP) or `make serve-http` (REST API)
- **Memory Validation**: Check `.memory_path` file and ensure entities/ directory exists

## Memory Connector Extensibility

All connectors inherit from `BaseMemoryConnector` in `memory_connectors/base.py`:
```python
class CustomConnector(BaseMemoryConnector):
    def extract_data(self, source_path: str) -> Dict[str, Any]: ...
    def organize_data(self, extracted_data: Dict[str, Any]) -> Dict[str, Any]: ...
    def generate_memory_files(self, organized_data: Dict[str, Any]) -> None: ...
```

### Supported Connectors
- **Export-based**: `chatgpt_history/`, `notion/`, `nuclino/` (process ZIP exports)
- **Live API**: `github_live/`, `google_docs_live/` (real-time data sync)

## Configuration Files & State

- **`.memory_path`**: Absolute path to memory directory (created by `make setup`)
- **`.mlx_model_name`**: MLX model variant for macOS (e.g., "mem-agent-mlx-4bit")
- **`.filters`**: Privacy/content filtering rules applied to agent responses
- **`agent/system_prompt.txt`**: Core agent behavior and response format rules

## Integration Patterns

### MCP Client Configuration
```json
// claude_desktop.json
{
  "mcpServers": {
    "memory-agent": {
      "command": "uvx",
      "args": ["--from", "/path/to/mem-agent-mcp", "mcp_server"]
    }
  }
}
```

### Memory Entity Relationships
```markdown
# user.md
- relationships:
  - company: [[entities/acme_corp.md]]

# entities/acme_corp.md  
- employees: [[entities/john_doe.md]]
```

## Platform-Specific Considerations

- **Python 3.11+ required** (3.12+ not supported due to vLLM compatibility)
- **macOS**: MLX framework for Apple Silicon, model quantization support
- **Linux**: CUDA/ROCm for vLLM backend, full precision models
- **Windows**: CPU-only, relies on cloud APIs

When modifying core agent logic, always test response format compliance with the `<think><python><reply>` structure defined in `agent/system_prompt.txt`.
