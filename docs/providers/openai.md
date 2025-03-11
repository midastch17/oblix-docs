# OpenAI

OpenAI is a leading AI research laboratory that provides a range of powerful language models through its API. This page covers how to use OpenAI models with Oblix for intelligent model orchestration.

## Installation and Setup

Oblix is available exclusively for macOS at this time. To install:

1. Visit [oblix.ai](https://oblix.ai) to download the latest version
2. Follow the installation instructions on the website
3. Once installed, you can import the SDK in your Python projects

> **Note:** Currently, only macOS is supported. Windows and Linux versions are planned for future releases.

You'll need an OpenAI API key to use OpenAI models. You can get one from the [OpenAI API Dashboard](https://platform.openai.com/api-keys).

## Hooking OpenAI Models

```python
from oblix import OblixClient, ModelType

# Initialize client
client = OblixClient(oblix_api_key="your_oblix_api_key")

# Hook OpenAI model
await client.hook_model(
    model_type=ModelType.OPENAI,
    model_name="gpt-3.5-turbo",
    api_key="your_openai_api_key"
)
```

## Supported OpenAI Models

Oblix supports most OpenAI models, including:

### GPT-4 Series
- `gpt-4-turbo`: The most capable and up-to-date GPT-4 model with 128K context
- `gpt-4`: Original GPT-4 model with 8K context
- `gpt-4-vision`: GPT-4 model with vision capabilities

### GPT-3.5 Series
- `gpt-3.5-turbo`: Balanced model optimized for chat at a lower cost
- `gpt-3.5-turbo-16k`: Extended context version of GPT-3.5 Turbo

## Using OpenAI with Oblix

Oblix is designed for orchestration between local and cloud models. While it's technically possible to use just OpenAI models, you'll get the most value from Oblix when combining cloud capabilities with local models for more resilient and cost-effective AI applications.

### Hybrid Local-Cloud Setup with OpenAI

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

# Hook cloud OpenAI model
await client.hook_model(
    model_type=ModelType.OPENAI,
    model_name="gpt-3.5-turbo",
    api_key="your_openai_api_key"
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

You can customize OpenAI model settings when executing prompts:

```python
response = await client.execute(
    "Write a story about a robot who gains consciousness",
    model_id="openai:gpt-4",
    temperature=0.8,
    max_tokens=2000,
    top_p=0.95
)
```

### Supported Parameters

OpenAI models in Oblix support all standard OpenAI parameters:

| Parameter | Description | Default |
|-----------|-------------|---------|
| `temperature` | Controls randomness (0-2) | 0.7 |
| `max_tokens` | Maximum response length | model-dependent |
| `top_p` | Nucleus sampling parameter | 1.0 |
| `presence_penalty` | Penalizes repeated tokens | 0.0 |
| `frequency_penalty` | Penalizes frequent tokens | 0.0 |

## Performance Considerations

When using OpenAI models with Oblix, consider:

### Cost Optimization
- Use `gpt-3.5-turbo` for routine tasks
- Reserve `gpt-4` for complex reasoning and creative tasks
- Let Oblix's orchestration system automatically route to local models when appropriate

### Latency Management
- OpenAI API calls introduce network latency
- Response time increases with model size and response length
- Oblix's connectivity monitoring helps manage latency expectations

### Token Counting
- OpenAI models have token limits per request
- Oblix tracks token usage for monitoring purposes
- Consider context length for complex conversations

### macOS Specific Considerations
- Oblix is optimized for macOS system resource monitoring
- Ensures efficient orchestration between local and cloud models on Mac hardware
- Takes advantage of Metal-compatible GPUs when available

## Error Handling

Oblix provides robust error handling for OpenAI API interactions:

```python
try:
    response = await client.execute("Generate a complex analysis", model_id="openai:gpt-4")
except Exception as e:
    print(f"Error: {e}")
    # Oblix will automatically try to use a fallback model if available
```

## API Reference

See the [OblixClient API reference](../api-reference/oblix-client.md) for detailed information on all available methods.