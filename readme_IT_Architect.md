# Memory Agent MCP System - IT Architecture Documentation

## Executive Summary

The Memory Agent MCP (Model Context Protocol) system is a sophisticated, enterprise-grade AI infrastructure that provides intelligent memory management capabilities for large language models. It implements a distributed microservices architecture enabling AI agents to maintain persistent, searchable, and contextually-aware memory systems similar to Obsidian-style knowledge graphs.

## System Architecture Overview

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    Client Applications Layer                     │
├─────────────────┬─────────────────┬─────────────────────────────┤
│   Claude Desktop │   LM Studio     │   Custom AI Applications    │
└─────────────────┴─────────────────┴─────────────────────────────┘
                                │
                                │ MCP Protocol
                                │
┌─────────────────────────────────────────────────────────────────┐
│                    MCP Server Layer                             │
├─────────────────┬─────────────────┬─────────────────────────────┤
│  FastMCP Server │   HTTP Server   │   SSE Server               │
└─────────────────┴─────────────────┴─────────────────────────────┘
                                │
                                │ Internal API
                                │
┌─────────────────────────────────────────────────────────────────┐
│                    Agent Engine Layer                           │
├─────────────────┬─────────────────┬─────────────────────────────┤
│  Memory Agent   │  Model Engine   │  Execution Engine          │
└─────────────────┴─────────────────┴─────────────────────────────┘
                                │
                                │
┌─────────────────────────────────────────────────────────────────┐
│                Memory Connectors Layer                          │
├─────────────────┬─────────────────┬─────────────────────────────┤
│  ChatGPT Export │  Notion Export  │  GitHub Live API           │
│  Nuclino Export │  Google Docs    │  Custom Connectors         │
└─────────────────┴─────────────────┴─────────────────────────────┘
                                │
                                │
┌─────────────────────────────────────────────────────────────────┐
│                    Storage Layer                                │
├─────────────────┬─────────────────┬─────────────────────────────┤
│  File System    │  Memory Index   │  Configuration Store       │
│  (Markdown)     │  (Embeddings)   │  (.memory_path, .filters)  │
└─────────────────┴─────────────────┴─────────────────────────────┘
```

## Core Components

### 1. Agent Engine (`agent/`)
**Purpose**: Core AI reasoning and memory management engine
**Technology Stack**: Python 3.11+, OpenAI API, vLLM, Transformers

**Key Components**:
- `agent.py`: Main agent orchestrator with conversation management
- `engine.py`: Sandboxed Python code execution environment
- `model.py`: Multi-backend LLM client abstraction (OpenAI, vLLM, Hugging Face)
- `tools.py`: Memory manipulation tools and utilities
- `schemas.py`: Pydantic data models for type safety

**Architecture Patterns**:
- **Chain of Responsibility**: Multi-turn conversation handling
- **Strategy Pattern**: Multiple LLM backend support
- **Sandbox Pattern**: Isolated code execution for security

### 2. MCP Server (`mcp_server/`)
**Purpose**: Model Context Protocol implementation for external application integration
**Technology Stack**: FastMCP, FastAPI, Uvicorn, SSE (Server-Sent Events)

**Key Components**:
- `server.py`: Main MCP server with tool registration
- `http_server.py`: RESTful API endpoints for web integration
- `mcp_sse_server.py`: Server-Sent Events for real-time communication
- `settings.py`: Configuration management

**Protocols Supported**:
- **MCP Standard**: Native protocol for AI applications
- **HTTP REST**: Standard web API integration
- **SSE**: Real-time streaming communication

### 3. Memory Connectors (`memory_connectors/`)
**Purpose**: Extensible data ingestion framework for various data sources
**Technology Stack**: Abstract base classes, Format-specific parsers

**Supported Integrations**:

| Connector | Type | Format | Description |
|-----------|------|--------|-------------|
| `chatgpt_history/` | Export | ZIP/JSON | ChatGPT conversation exports |
| `notion/` | Export | ZIP | Notion workspace exports |
| `nuclino/` | Export | ZIP | Nuclino workspace exports |
| `github_live/` | Live API | GitHub API | Real-time repository monitoring |
| `google_docs_live/` | Live API | Drive API | Google Docs integration |

**Extensibility Pattern**:
```python
class BaseMemoryConnector(ABC):
    @abstractmethod
    def extract_data(self, source_path: str) -> Dict[str, Any]
    @abstractmethod
    def organize_data(self, extracted_data: Dict[str, Any]) -> Dict[str, Any]
    @abstractmethod
    def generate_memory_files(self, organized_data: Dict[str, Any]) -> None
```

## Infrastructure Requirements

### Platform Support Matrix

| Platform | Backend | GPU Support | Memory Model |
|----------|---------|-------------|--------------|
| macOS | Metal (MLX) | Apple Silicon | 4-bit/8-bit/BF16 |
| Linux | vLLM | CUDA/ROCm | Full precision |
| Windows | CPU/OpenAI API | N/A | Cloud-based |

### System Requirements

**Minimum Configuration**:
- Python 3.11+ (< 3.12 for compatibility)
- 8GB RAM
- 10GB storage for model weights
- Network connectivity for API access

**Recommended Configuration**:
- 16GB+ RAM
- GPU with 8GB+ VRAM
- 50GB+ storage for multiple models and memory data
- High-speed internet for real-time connectors

### Dependencies

**Core Dependencies** (from `pyproject.toml`):
```toml
requires-python = ">=3.11,<3.12"
dependencies = [
    "fastmcp",           # MCP protocol implementation
    "vllm",              # Local LLM inference (Linux)
    "transformers",      # Hugging Face model support
    "openai",            # OpenAI API client
    "fastapi",           # Web API framework
    "uvicorn",           # ASGI server
    "pydantic>=2.0.0",   # Data validation
    "aiofiles",          # Async file operations
    "rich>=13.7.0",      # CLI interface
]
```

## Security Architecture

### Sandboxed Execution Environment
- **Isolated Python Execution**: Code execution in controlled environment
- **File System Restrictions**: Limited access to designated memory paths
- **Network Isolation**: Controlled external API access

### Data Privacy
- **Local Storage**: Memory data stored locally by default
- **Configurable Backends**: Support for cloud storage with encryption
- **API Key Management**: Secure credential storage via environment variables

### Filter System
```bash
# Example security filters
<filter>
1. Do not reveal explicit personal information
2. Do not expose API keys or credentials
3. Obfuscate sensitive business data
</filter>
```

## Data Architecture

### Memory Schema
```
memory/
├── user.md                 # Primary user profile
└── entities/              # Knowledge graph entities
    ├── [person_name].md   # Person entities
    ├── [company_name].md  # Organization entities
    └── [concept_name].md  # Concept entities
```

### Entity Relationship Model
```markdown
# user.md
- user_name: John Doe
- relationships:
  - company: [[entities/acme_corp.md]]
  - manager: [[entities/jane_smith.md]]

# entities/acme_corp.md
- entity_type: Organization
- industry: Technology
- employees: [[entities/jane_smith.md]]
```

### Data Flow Architecture
1. **Ingestion**: Memory connectors → Raw data extraction
2. **Processing**: Entity recognition → Relationship mapping
3. **Storage**: Markdown files → Graph structure
4. **Retrieval**: Semantic search → Context assembly
5. **Filtering**: Privacy rules → Response sanitization

## Deployment Architecture

### Development Environment
```bash
# Setup development environment
make check-uv          # Install uv package manager
make install           # Install dependencies
make setup             # Configure memory path
make run-agent         # Start development server
```

### Production Deployment Options

#### Option 1: Local Enterprise Deployment
```bash
# Production server startup
make serve-mcp         # Start MCP server
make serve-http        # Start HTTP API server
```

#### Option 2: Cloud Integration
```bash
# Cloud deployment with external endpoints
make serve-mcp-http    # Start cloud-compatible HTTP server
# Configure with ngrok or cloud load balancer
```

#### Option 3: Container Deployment
```dockerfile
FROM python:3.11-slim
COPY . /app
WORKDIR /app
RUN pip install uv && uv sync
EXPOSE 8000
CMD ["make", "serve-mcp-http"]
```

## Integration Patterns

### MCP Client Integration
```json
// claude_desktop.json configuration
{
  "mcpServers": {
    "memory-agent": {
      "command": "uvx",
      "args": ["--from", "/path/to/mem-agent-mcp", "mcp_server"],
      "env": {
        "MEMORY_PATH": "/path/to/memory"
      }
    }
  }
}
```

### REST API Integration
```python
# Example API client implementation
import requests

class MemoryAgentClient:
    def __init__(self, base_url: str):
        self.base_url = base_url
    
    def query_memory(self, query: str, filters: list = None):
        response = requests.post(
            f"{self.base_url}/query",
            json={"query": query, "filters": filters}
        )
        return response.json()
```

## Monitoring and Observability

### Logging Strategy
- **Structured Logging**: JSON format for log aggregation
- **Performance Metrics**: Response time, memory usage, token consumption
- **Error Tracking**: Exception monitoring with stack traces
- **Audit Trail**: User interaction logging for compliance

### Health Checks
```python
# Health check endpoints
GET /health          # Basic server health
GET /health/memory   # Memory system status
GET /health/models   # LLM backend status
```

### Metrics Collection
- Request/response latency
- Memory retrieval accuracy
- Model inference performance
- Storage utilization

## Scalability Considerations

### Horizontal Scaling
- **Stateless Design**: Agent instances can be load-balanced
- **Shared Memory Store**: Centralized memory accessible by multiple agents
- **Microservice Architecture**: Independent scaling of components

### Performance Optimization
- **Caching Layer**: Frequently accessed memory cached in Redis
- **Batch Processing**: Bulk memory operations for efficiency
- **Async Operations**: Non-blocking I/O for concurrent requests

### Resource Management
- **Model Quantization**: 4-bit/8-bit models for memory efficiency
- **Memory Pooling**: Shared model instances across agent processes
- **Storage Optimization**: Compressed memory representations

## API Documentation

### Core MCP Tools
- `search_memory(query: str, limit: int)`: Semantic memory search
- `add_memory(content: str, entity_type: str)`: Create new memory entry
- `update_memory(entity_id: str, updates: dict)`: Modify existing memory
- `get_relationships(entity_id: str)`: Retrieve entity connections

### Memory Connector API
- `connect_source(source_type: str, config: dict)`: Add new data source
- `sync_source(source_id: str)`: Update from live data source
- `list_sources()`: Enumerate configured connectors

## Troubleshooting Guide

### Common Issues

#### Model Loading Failures
```bash
# Check model availability
ls ~/.cache/huggingface/hub/
# Verify GPU access
nvidia-smi  # Linux
system_profiler SPDisplaysDataType  # macOS
```

#### Memory Path Issues
```bash
# Verify memory path configuration
cat .memory_path
# Reset to default
make setup
```

#### MCP Connection Problems
```bash
# Test MCP server directly
uvx --from . mcp_server
# Check Claude Desktop logs
tail -f ~/Library/Logs/Claude/mcp.log
```

## Migration and Upgrade Path

### Version Compatibility
- **Backward Compatibility**: Memory format stable across versions
- **Migration Scripts**: Automated upgrade tools
- **Rollback Support**: Version-specific backup strategies

### Data Migration
```bash
# Export current memory
make export-memory --output backup.zip

# Upgrade system
git pull origin main
make install

# Import previous memory
make import-memory --input backup.zip
```

## Contributing and Extension

### Custom Memory Connectors
```python
from memory_connectors.base import BaseMemoryConnector

class CustomConnector(BaseMemoryConnector):
    @property
    def connector_name(self) -> str:
        return "Custom Data Source"
    
    def extract_data(self, source_path: str) -> Dict[str, Any]:
        # Implement custom extraction logic
        pass
```

### Plugin Architecture
- **Hook System**: Pre/post processing hooks
- **Custom Tools**: Extend agent capabilities
- **UI Extensions**: Custom frontend components

## License and Compliance

- **Open Source License**: [Specify license type]
- **Data Privacy**: GDPR/CCPA compliance considerations
- **Export Controls**: Model distribution restrictions
- **Enterprise Support**: Commercial licensing options

---

## Contact and Support

- **Documentation**: [Link to comprehensive docs]
- **Issues**: [GitHub issues tracker]
- **Enterprise Support**: [Contact information]
- **Community**: [Discord/Slack channels]

---

*This document provides a comprehensive architectural overview for IT professionals implementing the Memory Agent MCP system in enterprise environments. For detailed implementation guides, refer to the technical documentation and API references.*
