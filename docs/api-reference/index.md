# API Reference

This section provides detailed documentation of the Oblix SDK API for orchestrating AI models between cloud and edge environments.

## Overview

Oblix provides a simple, unified API to orchestrate AI models across different providers (OpenAI, Claude, Ollama) with intelligent routing based on system resources and connectivity. The API is designed to be intuitive and developer-friendly while providing powerful orchestration capabilities.

## Main Components

### [OblixClient](oblix-client.md)

The main entry point to the SDK. Handles model management, orchestration, execution routing, and session management.

```python
from oblix import OblixClient

# Initialize with your API key
client = OblixClient(oblix_api_key="your_api_key")
```

### [ModelType](model-types.md)

Enum that defines the supported model types/providers for orchestration.

```python
from oblix import ModelType

# Available model types for orchestration
ModelType.OLLAMA  # Local edge models via Ollama
ModelType.OPENAI  # Cloud models from OpenAI (GPT series)
ModelType.CLAUDE  # Cloud models from Anthropic (Claude series)
```

### [Agents](agents.md)

Monitoring agents that provide system awareness for orchestration between cloud and edge.

```python
from oblix.agents import ResourceMonitor, ConnectivityAgent

# Add system monitoring for orchestration
client.hook_agent(ResourceMonitor())
client.hook_agent(ConnectivityAgent())
```

## Orchestration Workflow

Using the Oblix API for orchestration typically follows this pattern:

1. **Initialize client**
2. **Hook models** (at least one local edge model and one cloud model)
3. **Add monitoring agents** for orchestration
4. **Execute prompts** with automatic orchestration between cloud and edge

```python
import asyncio
from oblix import OblixClient, ModelType
from oblix.agents import ResourceMonitor, ConnectivityAgent

async def main():
    # 1. Initialize client
    client = OblixClient(oblix_api_key="your_api_key")
    
    # 2. Hook models for orchestration
    await client.hook_model(ModelType.OLLAMA, "llama2")  # Local edge model
    await client.hook_model(ModelType.OPENAI, "gpt-3.5-turbo", api_key="your_openai_api_key")  # Cloud model
    
    # 3. Add monitoring agents for orchestration
    client.hook_agent(ResourceMonitor())  # Monitors local resources
    client.hook_agent(ConnectivityAgent())  # Monitors network connectivity
    
    # 4. Execute a prompt with automatic orchestration
    response = await client.execute("Explain quantum computing in simple terms")
    print(f"Response from {response['model_id']}: {response['response']}")
    
    # 5. Access orchestration decision data
    agent_checks = response["agent_checks"]
    print(f"Resource state: {agent_checks.get('resource_monitor', {}).get('state')}")
    print(f"Connectivity: {agent_checks.get('connectivity_agent', {}).get('state')}")

# Run the example
if __name__ == "__main__":
    asyncio.run(main())
```

## Orchestration Benefits

Oblix's API provides several key benefits for AI orchestration:

1. **Automatic model selection** based on current device and network conditions
2. **Seamless fallback** between cloud and edge models
3. **Optimized performance** by balancing local computation and cloud capabilities
4. **Cost efficiency** by using local models when possible
5. **Resilience** against connectivity issues or resource constraints

## Next Steps

Explore detailed documentation for each component:

- [OblixClient API Reference](oblix-client.md)
- [ModelType Enum Reference](model-types.md)
- [Agents Reference](agents.md)