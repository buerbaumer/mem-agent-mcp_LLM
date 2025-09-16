# Memory Agent MCP System - Data Flow Analysis

## Overview

This document analyzes the complete data flow for users interacting with the Memory Agent MCP system, from initial query to final response, including all data transformations, storage operations, and system interactions.

## High-Level Data Flow

```mermaid
graph TB
    %% User Input Layer
    User[👤 User] -->|Query| Client[🖥️ Client Application]
    Client --> |MCP Protocol| MCPServer[🔌 MCP Server]
    
    %% Client Types
    Client --> Claude[Claude Desktop]
    Client --> LMStudio[LM Studio]
    Client --> ChatGPT[ChatGPT via HTTP]
    Client --> CLI[Chat CLI]
    
    %% MCP Server Layer
    MCPServer --> |use_memory_agent()| Agent[🧠 Memory Agent]
    
    %% Agent Processing
    Agent --> |1. Parse Query| Parser[📝 Query Parser]
    Agent --> |2. Apply Filters| FilterEngine[🔒 Filter Engine]
    Agent --> |3. Generate Response| LLM[🤖 LLM Backend]
    
    %% LLM Backend Options
    LLM --> OpenAI[OpenAI API]
    LLM --> vLLM[vLLM Server]
    LLM --> MLX[MLX Models]
    
    %% Memory Operations
    Agent --> |4. Execute Code| Sandbox[⚡ Sandboxed Engine]
    Sandbox --> |Memory Operations| Tools[🛠️ Memory Tools]
    
    %% Storage Layer
    Tools --> |Read/Write| FileSystem[💾 File System]
    FileSystem --> UserMD[user.md]
    FileSystem --> EntitiesDir[entities/]
    
    %% Response Flow
    Sandbox --> |Results| Agent
    Agent --> |Final Response| MCPServer
    MCPServer --> |MCP Protocol| Client
    Client --> |Display| User
    
    %% Configuration Files
    ConfigFiles[⚙️ Config Files] --> Agent
    ConfigFiles --> MemoryPath[.memory_path]
    ConfigFiles --> Filters[.filters]
    ConfigFiles --> ModelName[.mlx_model_name]
```

## Detailed Data Flow Stages

### 1. User Query Initiation

```mermaid
sequenceDiagram
    participant U as User
    participant C as Client App
    participant M as MCP Server
    participant A as Memory Agent
    
    U->>C: Types query: "What do you remember about my work?"
    C->>M: MCP Request: use_memory_agent()
    M->>A: Initialize Agent instance
    Note over A: Reads .memory_path, .filters, .mlx_model_name
    A->>A: Load system prompt
    A->>A: Create conversation history
```

### 2. Query Processing & LLM Interaction

```mermaid
sequenceDiagram
    participant A as Agent
    participant F as Filter Engine
    participant L as LLM Backend
    participant P as Parser
    
    A->>F: Apply privacy filters from .filters
    F->>A: Modified query with <filter> tags
    A->>L: Send messages to LLM
    L->>A: Raw LLM response
    A->>P: Extract <think>, <python>, <reply> sections
    P->>A: Structured response components
```

### 3. Memory Operations in Sandbox

```mermaid
sequenceDiagram
    participant A as Agent
    participant S as Sandbox Engine
    participant T as Memory Tools
    participant FS as File System
    
    A->>S: Execute Python code from LLM response
    S->>S: Change working directory to memory_path
    S->>S: Apply file access restrictions
    S->>T: Call memory functions (read_file, write_to_file, etc.)
    T->>FS: File operations within memory directory
    FS->>T: File contents / operation results
    T->>S: Function return values
    S->>A: Execution results + any errors
```

### 4. Memory Structure & Entity Management

```mermaid
graph LR
    subgraph "Memory Directory Structure"
        MemDir[memory/]
        UserFile[user.md]
        EntDir[entities/]
        
        MemDir --> UserFile
        MemDir --> EntDir
        
        subgraph "Entity Files"
            PersonA[john_doe.md]
            CompanyA[acme_corp.md]
            ConceptA[project_x.md]
        end
        
        EntDir --> PersonA
        EntDir --> CompanyA
        EntDir --> ConceptA
    end
    
    subgraph "Entity Relationships"
        UserFile -.->|[[entities/acme_corp.md]]| CompanyA
        UserFile -.->|[[entities/john_doe.md]]| PersonA
        CompanyA -.->|[[entities/john_doe.md]]| PersonA
    end
```

### 5. Response Generation & Multi-Turn Processing

```mermaid
sequenceDiagram
    participant A as Agent
    participant L as LLM
    participant S as Sandbox
    participant U as User
    
    Note over A: Initial response processing
    A->>L: Send query + conversation history
    L->>A: Response with <python> code
    A->>S: Execute Python code
    S->>A: Memory operation results
    
    Note over A: Multi-turn loop (up to MAX_TOOL_TURNS=20)
    loop Until <reply> received or max turns
        A->>A: Add results to conversation
        A->>L: Continue conversation
        L->>A: Next response
        alt Has Python code
            A->>S: Execute code
            S->>A: Results
        else Has reply
            A->>U: Return final response
        end
    end
```

## Data Transformations

### 1. Query Filter Application

```mermaid
graph LR
    OriginalQuery[Original Query] --> FilterEngine[Filter Engine]
    FiltersFile[.filters file] --> FilterEngine
    FilterEngine --> ModifiedQuery["Query + <filter> tags"]
    
    subgraph "Filter Examples"
        F1["1. Do not reveal personal info"]
        F2["2. Obfuscate business data"]
        F3["3. Hide contact details"]
    end
    
    FiltersFile -.-> F1
    FiltersFile -.-> F2
    FiltersFile -.-> F3
```

### 2. LLM Response Parsing

```mermaid
graph TD
    RawResponse[Raw LLM Response] --> Parser[Response Parser]
    
    Parser --> Think["<think>...</think>"]
    Parser --> Python["<python>...</python>"]
    Parser --> Reply["<reply>...</reply>"]
    
    Think --> ReasoningExtracted[Agent Reasoning]
    Python --> CodeExtracted[Python Code for Execution]
    Reply --> FinalResponse[Final User Response]
```

### 3. Memory File Operations

```mermaid
graph LR
    subgraph "Read Operations"
        ReadFile[read_file()] --> FileContent[File Content]
        FollowLink[follow_link()] --> LinkedContent[Linked File Content]
        ListFiles[list_files()] --> FileList[Directory Listing]
    end
    
    subgraph "Write Operations"
        WriteFile[write_to_file()] --> NewFile[Create/Update File]
        UpdateFile[update_file()] --> ModifiedFile[Modified File]
        CreateDir[create_directory()] --> NewDir[New Directory]
    end
    
    subgraph "Entity Link Resolution"
        ObsidianLink["[[entities/name.md]]"] --> FilePath[entities/name.md]
        FilePath --> FileAccess[File Read/Write]
    end
```

## Memory Connector Data Flow

```mermaid
graph TB
    subgraph "Data Source Integration"
        Sources[Data Sources] --> Connectors[Memory Connectors]
        
        Sources --> ChatGPTZip[ChatGPT Export .zip]
        Sources --> NotionZip[Notion Export .zip]
        Sources --> GitHubAPI[GitHub Live API]
        Sources --> GoogleDocs[Google Docs API]
    end
    
    subgraph "Connector Processing Pipeline"
        Connectors --> Extract[extract_data()]
        Extract --> Organize[organize_data()]
        Organize --> Generate[generate_memory_files()]
    end
    
    subgraph "Memory Output"
        Generate --> UserMD[user.md]
        Generate --> EntityFiles[entities/*.md]
        Generate --> Relationships[Entity Links]
    end
    
    EntityFiles --> MemorySystem[Memory Agent System]
    UserMD --> MemorySystem
    Relationships --> MemorySystem
```

## Platform-Specific Data Flows

### macOS with MLX Models

```mermaid
graph LR
    User --> MCPServer
    MCPServer --> Agent
    Agent --> |Read .mlx_model_name| ModelConfig[Model Configuration]
    ModelConfig --> MLXClient[MLX Client]
    MLXClient --> |4-bit/8-bit quantization| MLXModel[Local MLX Model]
    MLXModel --> |Inference| Response
    Response --> Agent
```

### Linux with vLLM

```mermaid
graph LR
    User --> MCPServer
    MCPServer --> Agent
    Agent --> vLLMClient[vLLM Client]
    vLLMClient --> |HTTP Request| vLLMServer[vLLM Server]
    vLLMServer --> |GPU Inference| LocalModel[Local LLM]
    LocalModel --> |Full Precision| Response
    Response --> Agent
```

### Cloud API Integration

```mermaid
graph LR
    User --> MCPServer
    MCPServer --> Agent
    Agent --> OpenAIClient[OpenAI Client]
    OpenAIClient --> |HTTPS| OpenRouter[OpenRouter API]
    OpenRouter --> |Cloud Model| Claude[Claude/GPT-4]
    Claude --> |API Response| Response
    Response --> Agent
```

## Error Handling & Recovery

```mermaid
graph TD
    Operation[Memory Operation] --> Success{Success?}
    
    Success -->|Yes| NormalFlow[Continue Normal Flow]
    Success -->|No| ErrorType{Error Type?}
    
    ErrorType -->|File Not Found| CreateFile[Create Missing File/Directory]
    ErrorType -->|Permission Denied| SecurityError[Return Security Error]
    ErrorType -->|Sandbox Timeout| TimeoutError[Return Timeout Error]
    ErrorType -->|LLM Error| RetryLLM[Retry with Fallback Model]
    ErrorType -->|Memory Limit| SizeError[Return Size Limit Error]
    
    CreateFile --> NormalFlow
    SecurityError --> ErrorResponse[Error Response to User]
    TimeoutError --> ErrorResponse
    RetryLLM --> NormalFlow
    SizeError --> ErrorResponse
```

## Performance Considerations

### Memory Access Patterns

```mermaid
graph LR
    subgraph "Hot Path (Frequent Access)"
        UserFile[user.md]
        RecentEntities[Recent entities/*.md]
        ConfigFiles[.memory_path, .filters]
    end
    
    subgraph "Cold Path (Infrequent Access)"
        ArchiveEntities[Archive entities/*.md]
        LargeFiles[Large content files]
        HistoricalData[Historical conversation data]
    end
    
    Agent --> |High Frequency| UserFile
    Agent --> |Medium Frequency| RecentEntities
    Agent --> |Low Frequency| ArchiveEntities
```

### Caching Strategy

```mermaid
graph TD
    Request[Memory Request] --> Cache{In Cache?}
    
    Cache -->|Hit| ServeFromCache[Serve from Cache]
    Cache -->|Miss| ReadFromDisk[Read from File System]
    
    ReadFromDisk --> UpdateCache[Update Cache]
    UpdateCache --> ServeResponse[Serve Response]
    ServeFromCache --> Response[Response to Agent]
    ServeResponse --> Response
```

## Security Data Flow

```mermaid
graph TD
    UserQuery[User Query] --> FilterCheck[Privacy Filter Check]
    FilterCheck --> SandboxEntry[Enter Sandbox Environment]
    
    SandboxEntry --> PathValidation[Validate File Paths]
    PathValidation --> MemoryBounds{Within Memory Directory?}
    
    MemoryBounds -->|Yes| AllowedOperation[Execute Operation]
    MemoryBounds -->|No| DenyAccess[Deny Access]
    
    AllowedOperation --> SizeCheck[Check Size Limits]
    SizeCheck --> ExecuteCode[Execute Memory Code]
    
    ExecuteCode --> FilterResponse[Apply Response Filters]
    FilterResponse --> SecureResponse[Filtered Response to User]
    
    DenyAccess --> SecurityError[Return Security Error]
```

## Configuration Data Flow

```mermaid
graph TB
    Startup[System Startup] --> LoadConfig[Load Configuration]
    
    LoadConfig --> ReadMemoryPath[Read .memory_path]
    LoadConfig --> ReadFilters[Read .filters]
    LoadConfig --> ReadModelName[Read .mlx_model_name]
    
    ReadMemoryPath --> ValidatePath{Path Valid?}
    ValidatePath -->|Yes| SetMemoryDir[Set Memory Directory]
    ValidatePath -->|No| DefaultPath[Use Default Path]
    
    ReadFilters --> ApplyFilters[Apply Privacy Filters]
    ReadModelName --> SelectModel[Select LLM Model]
    
    SetMemoryDir --> AgentReady[Agent Ready]
    DefaultPath --> AgentReady
    ApplyFilters --> AgentReady
    SelectModel --> AgentReady
```

---

*This data flow analysis provides a comprehensive view of how information moves through the Memory Agent MCP system, from user input to persistent storage and response generation. Understanding these flows is crucial for debugging, optimization, and extending the system's capabilities.*
