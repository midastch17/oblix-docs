# Anthropic Claude

Anthropic is an AI safety and research company, and is the creator of `Claude`. This page covers how to use Claude models with Oblix for intelligent model orchestration.

## Installation and Setup

Oblix is available exclusively for macOS at this time:

1. Visit [oblix.ai](https://oblix.ai) to download the latest version
2. Follow the installation instructions on the website
3. Once installed, you can import the SDK in your Python projects

> **Note:** Currently, only macOS is supported. Windows and Linux versions are planned for future releases.

You'll need an Anthropic API key to use Claude models with Oblix. You can obtain one from the [Anthropic Console](https://console.anthropic.com/).

## Hooking Claude Models

```python
from oblix import OblixClient, ModelType

# Initialize client
client = OblixClient(oblix_api_key="your_oblix_api_key")

# Hook Claude model
await client.hook_model(
    model_type=ModelType.CLAUDE,
    model_name="claude-3-opus-20240229",
    api_key="your_anthropic_api_key"
)
```

## Supported Claude Models

Oblix supports all current Claude models:

- `claude-3-opus-20240229` - Most powerful Claude model with 200K context window
- `claude-3-sonnet-20240229` - Balanced performance Claude model with 200K context window
- `claude-3-haiku-20240307` - Fastest Claude model with 200K context window

## Using Claude with Oblix

Oblix is designed for orchestration between local and cloud models. While Claude models are powerful, you'll get the most value from Oblix when combining them with local models in a hybrid setup for better resilience, latency, and cost management.

### Hybrid Local-Cloud Setup with Claude

```python
from oblix import OblixClient, ModelType
from oblix.agents.resource_monitor import ResourceMonitor
from oblix.agents.connectivity import ConnectivityAgent

# Initialize client
client = OblixClient(oblix_api_key="your_oblix_api_key")

# Hook local Ollama model
await client.hook_model(
    model_type=ModelType.OLLAMA,
    model_name="llama2",
    endpoint="http://localhost:11434"
)

# Hook cloud Claude model
await client.hook_model(
    model_type=ModelType.CLAUDE,
    model_name="claude-3-haiku-20240307",
    api_key="your_anthropic_api_key"
)

# Add monitoring agents for intelligent orchestration
client.hook_agent(ResourceMonitor())
client.hook_agent(ConnectivityAgent())

# Execute prompt - Oblix will automatically choose between 
# local and cloud models based on connectivity and system resources
response = await client.execute("Explain quantum computing in simple terms")
print(response["response"])
```

## Advanced Configuration

You can customize Claude model settings when executing prompts:

```python
response = await client.execute(
    "Write a story about a robot who gains consciousness",
    temperature=0.8,
    max_tokens=2000
)
```

### Supported Parameters

Claude models in Oblix support all standard Anthropic parameters:

| Parameter | Description | Default |
|-----------|-------------|---------|
| `temperature` | Controls randomness (0-1) | 0.7 |
| `max_tokens` | Maximum response length | 4096 |
| `top_p` | Nucleus sampling parameter | 1.0 |

## Performance Considerations

When using Claude models with Oblix, consider:

### Model Selection
- Use `claude-3-haiku` for faster responses and routine tasks
- Use `claude-3-sonnet` for balanced performance and quality
- Reserve `claude-3-opus` for complex reasoning and highest quality outputs

### Orchestration Benefits
- Oblix's connectivity monitoring helps manage Claude API latency
- Automatic fallback to local models when connectivity is limited
- Intelligent model selection based on task complexity and resource availability

### macOS Integration
- Oblix is optimized specifically for macOS system resource monitoring
- The platform provides smooth integration between local Ollama models and cloud-based Claude models
- Takes advantage of macOS performance metrics for intelligent orchestration decisions

## Error Handling

Oblix provides robust error handling for Claude API interactions:

```python
try:
    response = await client.execute("Generate a complex analysis")
except Exception as e:
    print(f"Error: {e}")
    # Oblix will automatically try to use a fallback model if available
```

## API Reference

See the [OblixClient API reference](../api-reference/oblix-client.md) for detailed information on all available methods.