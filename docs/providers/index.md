# Model Providers

Oblix supports multiple AI model providers, allowing you to seamlessly switch between local and cloud models based on your needs. This page provides an overview of the supported providers and how to configure them.

## Supported Providers

Oblix currently supports the following model providers:

| Provider | Type | Description | Key Features |
|----------|------|-------------|--------------|
| [Ollama](ollama.md) | Local | Run open-source models locally | Privacy, offline capability, no usage fees |
| [OpenAI](openai.md) | Cloud | Access GPT models via API | High capability, extensive features |
| [Claude](claude.md) | Cloud | Access Anthropic's Claude models | Long context windows, reasoning capabilities |

## Provider Configuration

Each provider requires specific configuration when hooking models:

```python
from oblix import OblixClient, ModelType

client = OblixClient(oblix_api_key="your_oblix_api_key")

# Hook Ollama (local) model
await client.hook_model(
    model_type=ModelType.OLLAMA,
    model_name="llama2",
    endpoint="http://localhost:11434"  # Optional, this is the default
)

# Hook OpenAI (cloud) model
await client.hook_model(
    model_type=ModelType.OPENAI,
    model_name="gpt-3.5-turbo",
    api_key="your_openai_api_key"
)

# Hook Claude (cloud) model
await client.hook_model(
    model_type=ModelType.CLAUDE,
    model_name="claude-3-opus-20240229",
    api_key="your_anthropic_api_key"
)
```

## Multi-Provider Strategy

For optimal results, Oblix recommends configuring at least:

1. **One local model** (via Ollama) for offline capability and privacy-sensitive tasks
2. **One cloud model** (OpenAI or Claude) for more demanding tasks

This hybrid approach enables Oblix to intelligently route prompts based on:

- Current connectivity status
- System resource availability
- Task complexity requirements

## Provider Selection Process

When executing a prompt, Oblix selects the provider based on:

1. **Explicit selection**: If a specific model is requested
2. **Agent recommendations**: Based on resource and connectivity checks
3. **Default fallback**: Using the first available model

You can see which provider was used in the response:

```python
response = await client.execute("Explain quantum computing")
print(f"Used provider/model: {response['model_id']}")
```

## Adding Custom Providers

Oblix can be extended to support additional model providers by:

1. Creating a custom model implementation that extends `BaseModel`
2. Implementing required methods (initialize, generate, shutdown)
3. Registering the model with the appropriate `ModelType`

## Provider-Specific Documentation

For detailed information about each provider, including supported models, configuration options, and best practices, see:

- [Ollama](ollama.md) - Local models
- [OpenAI](openai.md) - GPT models
- [Claude](claude.md) - Claude models
