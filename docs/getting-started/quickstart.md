# Quickstart Guide

Get started with Oblix, an AI orchestration SDK for seamless switching between local and cloud models based on connectivity and system resources.

## Installation

1. Download the Oblix installer (.dmg) from [Oblix AI](https://oblixai.com/download)
2. Open the .dmg file
3. Drag the Oblix application to your Applications folder
4. Launch Oblix to complete the installation

**Note:** Oblix is currently available for macOS only.

## Basic Usage

### 1. Initialize Client

```python
from oblix import OblixClient

# Initialize the client with your Oblix API key
client = OblixClient(oblix_api_key="your_api_key")
```

### 2. Hook Models

Connect at least one local and one cloud model:

```python
from oblix import ModelType

# Hook a local Ollama model
await client.hook_model(
    model_type=ModelType.OLLAMA, 
    model_name="llama2"
)

# Hook a cloud model (OpenAI)
await client.hook_model(
    model_type=ModelType.OPENAI, 
    model_name="gpt-3.5-turbo", 
    api_key="your_openai_api_key"
)
```

### 3. Add Monitoring Agents

```python
from oblix.agents import ResourceMonitor, ConnectivityAgent

# Add resource monitoring
client.hook_agent(ResourceMonitor())

# Add connectivity monitoring
client.hook_agent(ConnectivityAgent())
```

### 4. Execute Prompts

```python
# Execute a prompt - Oblix automatically orchestrates the optimal model
response = await client.execute("Explain quantum computing in simple terms")
print(response["response"])
```

### 5. Interactive Chat Session

```python
# Start an interactive chat session
await client.start_chat()
```

## Example: Complete Hybrid Setup

```python
import asyncio
from oblix import OblixClient, ModelType
from oblix.agents import ResourceMonitor, ConnectivityAgent

async def main():
    # Initialize client
    client = OblixClient(oblix_api_key="your_oblix_api_key")
    
    # Hook models
    await client.hook_model(ModelType.OLLAMA, "llama2")
    await client.hook_model(ModelType.OPENAI, "gpt-3.5-turbo", api_key="your_openai_api_key")
    
    # Add monitoring agents
    client.hook_agent(ResourceMonitor())
    client.hook_agent(ConnectivityAgent())
    
    # Execute a prompt
    response = await client.execute("Explain quantum computing in simple terms")
    print(response["response"])
    
    # Start interactive chat
    await client.start_chat()

# Run the example
if __name__ == "__main__":
    asyncio.run(main())
```

## What's Next?

- Learn about [how Oblix works](../core-concepts/how-it-works.md)
- Explore [examples](../examples/basic-usage.md)
- Set up different [model providers](../providers/index.md)
- Check out the [API reference](../api-reference/oblix-client.md)