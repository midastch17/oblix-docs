# API Reference

This section provides detailed documentation of the Oblix SDK API.

## Overview

Oblix provides a simple, unified API to orchestrate AI models across different providers (OpenAI, Claude, Ollama) with intelligent routing based on system resources and connectivity. The API is designed to be intuitive and developer-friendly while providing powerful capabilities.

## Main Components

### [OblixClient](oblix-client.md)

The main entry point to the SDK. Handles model management, execution routing, and session management.

```python
from oblix import OblixClient

# Initialize with your API key
client = OblixClient(oblix_api_key="your_api_key")
```

### [ModelType](model-types.md)

Enum that defines the supported model types/providers.

```python
from oblix import ModelType

# Available model types
ModelType.OLLAMA  # Local models via Ollama
ModelType.OPENAI  # OpenAI models (GPT series)
ModelType.CLAUDE  # Anthropic Claude models
```

### [Agents](agents.md)

Monitoring agents that provide system awareness for intelligent routing.

```python
from oblix.agents import ResourceMonitor, ConnectivityAgent

# Add monitoring capabilities
client.hook_agent(ResourceMonitor())
client.hook_agent(ConnectivityAgent())
```

## Basic Workflow

Using the Oblix API typically follows this pattern:

1. **Initialize client**
2. **Hook models** (at least one local and one cloud model)
3. **Add monitoring agents**
4. **Execute prompts** or start chat sessions

```python
import asyncio
from oblix import OblixClient, ModelType
from oblix.agents import ResourceMonitor, ConnectivityAgent

async def main():
    # 1. Initialize client
    client = OblixClient(oblix_api_key="your_api_key")
    
    # 2. Hook models
    await client.hook_model(ModelType.OLLAMA, "llama2")
    await client.hook_model(ModelType.OPENAI, "gpt-3.5-turbo", api_key="your_openai_api_key")
    
    # 3. Add monitoring agents
    client.hook_agent(ResourceMonitor())
    client.hook_agent(ConnectivityAgent())
    
    # 4. Execute a prompt
    response = await client.execute("Explain quantum computing in simple terms")
    print(response["response"])

# Run the example
if __name__ == "__main__":
    asyncio.run(main())
```

## Next Steps

Explore detailed documentation for each component:

- [OblixClient API Reference](oblix-client.md)
- [ModelType Enum Reference](model-types.md)
- [Agents Reference](agents.md)


This index page gives a clear overview of the Oblix API, introduces the main components, and provides a basic workflow example. It also includes links to more detailed documentation for each API component.
