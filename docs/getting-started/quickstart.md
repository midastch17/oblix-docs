# Quickstart Guide

Get started with Oblix, an AI orchestration SDK for seamless switching between local and cloud models based on connectivity and system resources.

## Installation

1. Download the Oblix installer (.dmg) from [Oblix AI](https://oblixai.com/download)
2. Open the .dmg file
3. Drag the Oblix application to your Applications folder
4. Launch Oblix to complete the installation

**Note:** Oblix is currently available for macOS only.

## Prerequisites

Before using Oblix with local models, you'll need to:

1. Install [Ollama](https://ollama.ai/) on your machine
2. Download at least one model using Ollama (e.g., `ollama pull llama2`)

## Basic Usage

### 1. Initialize Client

```python
import os
from dotenv import load_dotenv
from oblix import OblixClient

# Load environment variables from .env file
load_dotenv()

# Initialize the client with your Oblix API key
oblix_api_key = os.getenv('OBLIX_API_KEY')
client = OblixClient(oblix_api_key=oblix_api_key)
```

### 2. Hook Models

Connect at least one local and one cloud model:

```python
from oblix import ModelType

# Hook a local Ollama model (replace "llama2" with any model you've pulled)
await client.hook_model(
    model_type=ModelType.OLLAMA, 
    model_name="llama2",  # Use any available Ollama model
    endpoint="http://localhost:11434"  # Default Ollama endpoint
)

# Hook a cloud model (OpenAI)
openai_api_key = os.getenv('OPENAI_API_KEY')
await client.hook_model(
    model_type=ModelType.OPENAI, 
    model_name="gpt-3.5-turbo", 
    api_key=openai_api_key
)
```

### 3. Add Monitoring Agents

```python
from oblix.agents import ResourceMonitor, ConnectivityAgent

# Add resource monitoring
resource_monitor = ResourceMonitor(name="resource_monitor")

# Add connectivity monitoring
connectivity_monitor = ConnectivityAgent(name="connectivity_monitor")

# Register agents with the client
client.hook_agent(resource_monitor)
client.hook_agent(connectivity_monitor)
```

### 4. Execute Prompts

```python
# Execute a prompt - Oblix automatically orchestrates the optimal model
response = await client.execute("Explain quantum computing in simple terms")
print(response["response"])
```

### 5. Interactive Streaming Chat Session

```python
# Start an interactive streaming chat session
await client.chat_streaming()

# When finished, clean up resources
await client.shutdown()
```

## Example: Complete Setup

```python
#!/usr/bin/env python3
import asyncio
import os
from dotenv import load_dotenv
from oblix import OblixClient, ModelType
from oblix.agents import ResourceMonitor, ConnectivityAgent

# Load environment variables from .env file
load_dotenv()

async def main():
    # Initialize client with API key from environment
    oblix_api_key = os.getenv('OBLIX_API_KEY')
    if not oblix_api_key:
        raise ValueError("OBLIX_API_KEY environment variable must be set")
    
    client = OblixClient(oblix_api_key=oblix_api_key)
    
    # Set up resource monitor
    resource_monitor = ResourceMonitor(name="resource_monitor")
    
    # Set up connectivity monitor
    connectivity_monitor = ConnectivityAgent(name="connectivity_monitor")
    
    # Hook models - local Ollama model (replace with any model you've pulled)
    await client.hook_model(
        model_type=ModelType.OLLAMA,
        model_name="llama2",  # Use any available Ollama model
        endpoint="http://localhost:11434"
    )
    
    # Hook models - cloud OpenAI model
    openai_api_key = os.getenv('OPENAI_API_KEY')
    if not openai_api_key:
        raise ValueError("OPENAI_API_KEY environment variable must be set")
    
    await client.hook_model(
        model_type=ModelType.OPENAI, 
        model_name="gpt-3.5-turbo", 
        api_key=openai_api_key
    )
    
    # Add monitoring agents (Oblix will use these for orchestration)
    client.hook_agent(resource_monitor)
    client.hook_agent(connectivity_monitor)
    
    # Start interactive streaming chat
    await client.chat_streaming()
    
    # Clean up resources
    await client.shutdown()

# Run the example
if __name__ == "__main__":
    try:
        asyncio.run(main())
    except KeyboardInterrupt:
        print("\nProgram interrupted by user. Exiting...")
    except ValueError as e:
        print(f"\nConfiguration Error: {str(e)}")
        print("Please ensure all required environment variables are set.")
    except Exception as e:
        print(f"\nError: {str(e)}")
        import traceback
        traceback.print_exc()
```

## What's Next?

- Learn about [how Oblix works](../core-concepts/how-it-works.md)
- Explore [examples](../examples/basic-usage.md)
- Set up different [model providers](../providers/index.md)
- Check out the [API reference](../api-reference/oblix-client.md)
