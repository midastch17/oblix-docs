# OblixClient API Reference

The `OblixClient` class is the main entry point for the Oblix SDK. It provides a high-level interface for working with multiple AI models, managing sessions, and utilizing intelligent orchestration based on system resources and connectivity.

## Initialization

```python
from oblix import OblixClient

client = OblixClient(
    host: str = "localhost",     # Host address (default: localhost)
    port: int = 4321,           # Port number (default: 4321)
    config_path: str = None     # Path to configuration file (optional)
)
```

> **Note:** Oblix no longer requires an API key for initialization.

## Core Methods

### Hook Model

Register a new AI model with the client for orchestration.

```python
async def hook_model(
    model_type: ModelType,      # Type of model (e.g., ModelType.OLLAMA)
    model_name: str,            # Name of model (e.g., "llama2", "gpt-3.5-turbo") 
    endpoint: str = None,       # API endpoint for Ollama models (optional)
    api_key: str = None,        # API key for cloud models (optional)
    **kwargs                    # Additional model-specific parameters
) -> bool:                      # Returns True if successful
```

Example:
```python
# Hook a local Ollama model
await client.hook_model(
    model_type=ModelType.OLLAMA,
    model_name="llama2",
    endpoint="http://localhost:11434"
)

# Hook an OpenAI model
await client.hook_model(
    model_type=ModelType.OPENAI,
    model_name="gpt-3.5-turbo",
    api_key="your_openai_api_key"
)

# Hook a Claude model
await client.hook_model(
    model_type=ModelType.CLAUDE,
    model_name="claude-3-opus-20240229",
    api_key="your_anthropic_api_key"
)
```

### Hook Agent

Register an agent with the client for orchestration monitoring.

```python
def hook_agent(
    agent: BaseAgent           # Agent instance to hook
) -> bool:                     # Returns True if successful
```

Example:
```python
from oblix.agents import ResourceMonitor, ConnectivityAgent

# Add resource monitoring for orchestration
client.hook_agent(ResourceMonitor())

# Add connectivity monitoring for orchestration
client.hook_agent(ConnectivityAgent(
    latency_threshold=150.0,       # Maximum acceptable latency in ms (default: 200.0)
    packet_loss_threshold=5.0,     # Maximum acceptable packet loss percentage (default: 10.0)
    bandwidth_threshold=10.0       # Minimum acceptable bandwidth in Mbps (default: 5.0)
))
```

### Execute

Execute a prompt using available models with intelligent orchestration.

```python
async def execute(
    prompt: str,               # User prompt to process
    temperature: float = None, # Sampling temperature (optional)
    max_tokens: int = None,    # Maximum tokens to generate (optional)
    request_id: str = None,    # Custom request identifier (optional)
    **kwargs                   # Additional model-specific parameters
) -> dict:                     # Response dictionary
```

Response structure:
```python
{
    "request_id": str,         # Request identifier
    "model_id": str,           # ID of model used for generation
    "response": str,           # Generated text response
    "metrics": dict,           # Performance metrics
    "routing_decision": dict,  # Orchestration decisions
    "agent_checks": list       # Results from agent checks (list of dicts)
}
```

Example:
```python
# Execution with automatic orchestration
response = await client.execute("Explain quantum computing")
print(response["response"])
print(f"Model used by orchestration: {response['model_id']}")
```

> **Note:** For Oblix to provide orchestration, you should hook at least one local model (Ollama), one cloud model (OpenAI/Claude), and the appropriate monitoring agents. Orchestration decisions are made automatically based on resource and connectivity policies.

### Execute Streaming

Execute a prompt with streaming output.

```python
async def execute_streaming(
    prompt: str,               # User prompt to process
    temperature: float = None, # Sampling temperature (optional)
    max_tokens: int = None,    # Maximum tokens to generate (optional)
    request_id: str = None,    # Custom request identifier (optional)
    display_metrics: bool = True, # Whether to display metrics (optional)
    **kwargs                   # Additional model-specific parameters
) -> dict:                     # Final response dictionary
```

Example:
```python
# Stream a response with automatic orchestration
response = await client.execute_streaming("Explain quantum computing")
# Tokens are printed to the console in real-time, then final response returned
print(f"Model used by orchestration: {response['model_id']}")
```

### Chat Once

Send a single message in a chat session with automatic session handling.

```python
async def chat_once(
    prompt: str,               # User message
    session_id: str = None     # Existing session ID (optional)
) -> dict:                     # Response dictionary including session_id
```

Example:
```python
# Send a message (creates a new session)
response = await client.chat_once("Hello, how are you?")
session_id = response["session_id"]

# Continue the conversation
response = await client.chat_once("Tell me about yourself", session_id)
```

### Start Interactive Chat

Start an interactive chat session in the terminal with orchestration.

```python
async def start_chat(
    session_id: str = None     # Optional existing session ID to resume
) -> str:                      # Session ID for the current chat
```

Example:
```python
# Start a new interactive chat session
await client.start_chat()

# Resume an existing session
await client.start_chat("5f3e9a7b-6c1d-4d8e-9e7a-9b8c7d6e5f4a")
```

### Start Interactive Chat with Streaming

Start an interactive chat session with streaming responses.

```python
async def chat_streaming(
    session_id: str = None     # Optional existing session ID to resume
) -> str:                      # Session ID for the current chat
```

Example:
```python
# Start a new interactive streaming chat session
await client.chat_streaming()

# Resume an existing session with streaming
await client.chat_streaming("5f3e9a7b-6c1d-4d8e-9e7a-9b8c7d6e5f4a")
```

## Session Management Methods

### Create Session

Create a new chat session.

```python
async def create_session(
    title: str = None,           # Optional session title
    initial_context: dict = None, # Optional initial context
    metadata: dict = None        # Optional metadata for categorization
) -> str:                        # New session ID
```

### Create and Use Session

Create a new chat session and automatically set it as the current session.

```python
async def create_and_use_session(
    title: str = None,           # Optional session title
    initial_context: dict = None, # Optional initial context
    metadata: dict = None        # Optional metadata for categorization
) -> str:                        # New session ID (already set as current)
```

### Use Session

Set an existing session as the current active session.

```python
def use_session(
    session_id: str              # Session identifier
) -> bool:                       # True if session was activated
```

### List Sessions

List recent chat sessions with optional metadata filtering.

```python
def list_sessions(
    limit: int = 50,             # Maximum number of sessions to return
    filter_metadata: dict = None # Optional filter by metadata fields
) -> list:                       # List of session metadata dictionaries
```

### Load Session

Load a specific chat session by ID.

```python
def load_session(
    session_id: str              # Session identifier
) -> dict:                       # Session data if found, None otherwise
```

### Delete Session

Delete a chat session permanently.

```python
def delete_session(
    session_id: str              # Session identifier
) -> bool:                       # True if deleted successfully
```

### Update Session Metadata

Update or add metadata to a session.

```python
def update_session_metadata(
    session_id: str,             # Session identifier
    metadata: dict               # Metadata fields to update or add
) -> bool:                       # True if metadata was updated
```

### Get Session Metadata

Get metadata for a specific session without loading the entire content.

```python
def get_session_metadata(
    session_id: str              # Session identifier
) -> dict:                       # Session metadata if found
```

### Export Session

Export a session to a file for sharing or backup.

```python
async def export_session(
    session_id: str,             # Session identifier
    export_path: str             # Path to save the exported session
) -> bool:                       # True if export was successful
```

### Import Session

Import a session from a file.

```python
async def import_session(
    import_path: str,            # Path to the JSON file to import
    new_id: bool = True,         # Assign a new ID to avoid conflicts
    use_immediately: bool = False # Set as current session after import
) -> str:                        # Session ID of the imported session
```

### Copy Session

Create a copy of an existing session with a new ID.

```python
async def copy_session(
    session_id: str,             # Session identifier to copy
    new_title: str = None,       # Optional new title for the copy
    use_immediately: bool = False # Set as current session after copy
) -> str:                        # ID of the new copy
```

### Merge Sessions

Combine multiple sessions into a new session with unified history.

```python
async def merge_sessions(
    source_ids: list,            # List of session IDs to merge
    title: str = None,           # Optional title for merged session
    use_immediately: bool = False # Set as current session after merge
) -> str:                        # ID of the merged session
```

## Monitoring and Orchestration Methods

### List Models

List all available models grouped by type.

```python
def list_models() -> dict:    # Dictionary mapping model types to lists of model names
```

### Get Model

Get configuration for a specific model.

```python
def get_model(
    model_type: str,          # Type of model (e.g., 'ollama', 'openai', 'claude')
    model_name: str           # Name of model
) -> dict:                    # Model configuration if found
```

### Get Resource Metrics

Get current resource metrics from the resource monitor agent.

```python
async def get_resource_metrics() -> dict:  # Dictionary of resource metrics
```

### Get Connectivity Metrics

Get current connectivity metrics from the connectivity monitor.

```python
async def get_connectivity_metrics() -> dict:  # Dictionary of connectivity metrics
```

Example:
```python
# Check current connectivity for cloud model viability
connectivity = await client.get_connectivity_metrics()
print(f"Current latency: {connectivity['latency']}ms")

# Check resource availability for local model execution
resources = await client.get_resource_metrics()
print(f"CPU usage: {resources['cpu_percent']}%")
```

### Shutdown

Gracefully shut down all models and agents.

```python
async def shutdown() -> None:  # Clean up all resources
```