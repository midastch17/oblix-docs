# Basic Usage

This page provides basic examples to help you get started with Oblix. These examples cover installation, initialization, model hooking, and executing prompts with the SDK.

## Installation

Oblix is available exclusively for macOS at this time:

1. Download the Oblix installer (.dmg) from [Oblix AI](https://oblixai.com/download)
2. Open the .dmg file
3. Drag the Oblix application to your Applications folder
4. Launch Oblix to complete the installation

> **Note:** Currently, only macOS is supported. Windows and Linux versions are planned for future releases.

## Prerequisites

Before using Oblix with local models, you'll need to:

1. Install [Ollama](https://ollama.ai/) on your machine
2. Download at least one model using Ollama (e.g., `ollama pull llama2`)

## Initialization

Initialize the Oblix client with your API key:

```python
import asyncio
import os
from dotenv import load_dotenv
from oblix import OblixClient

async def main():
    # Load environment variables from .env file
    load_dotenv()
    
    # Initialize with API key from environment
    oblix_api_key = os.getenv('OBLIX_API_KEY')
    client = OblixClient(oblix_api_key=oblix_api_key)
    
    # Rest of your code here...
    
    # Clean up resources when done
    await client.shutdown()

# Run the async function
if __name__ == "__main__":
    asyncio.run(main())
```

## Hooking Models

Add models to your client for execution:

```python
import asyncio
import os
from dotenv import load_dotenv
from oblix import OblixClient, ModelType

async def main():
    # Load environment variables from .env file
    load_dotenv()
    
    oblix_api_key = os.getenv('OBLIX_API_KEY')
    client = OblixClient(oblix_api_key=oblix_api_key)
    
    # Hook a local Ollama model (replace with any model you've pulled)
    await client.hook_model(
        model_type=ModelType.OLLAMA,
        model_name="llama2",  # Use any available Ollama model
        endpoint="http://localhost:11434"  # Default Ollama endpoint
    )
    
    # Hook an OpenAI model
    openai_api_key = os.getenv('OPENAI_API_KEY')
    await client.hook_model(
        model_type=ModelType.OPENAI,
        model_name="gpt-3.5-turbo",
        api_key=openai_api_key
    )
    
    # Hook a Claude model
    anthropic_api_key = os.getenv('ANTHROPIC_API_KEY')
    await client.hook_model(
        model_type=ModelType.CLAUDE,
        model_name="claude-3-opus-20240229",
        api_key=anthropic_api_key
    )
    
    # Clean up resources when done
    await client.shutdown()

if __name__ == "__main__":
    asyncio.run(main())
```

## Adding Monitoring Agents

Add monitoring agents to enable intelligent orchestration:

```python
import asyncio
import os
from dotenv import load_dotenv
from oblix import OblixClient, ModelType
from oblix.agents import ResourceMonitor, ConnectivityAgent

async def main():
    # Load environment variables from .env file
    load_dotenv()
    
    oblix_api_key = os.getenv('OBLIX_API_KEY')
    client = OblixClient(oblix_api_key=oblix_api_key)
    
    # Hook models
    await client.hook_model(
        ModelType.OLLAMA, 
        "llama2",
        endpoint="http://localhost:11434"
    )
    
    openai_api_key = os.getenv('OPENAI_API_KEY')
    await client.hook_model(
        ModelType.OPENAI, 
        "gpt-3.5-turbo", 
        api_key=openai_api_key
    )
    
    # Add resource monitoring
    resource_monitor = ResourceMonitor(name="resource_monitor")
    client.hook_agent(resource_monitor)
    
    # Add connectivity monitoring
    connectivity_monitor = ConnectivityAgent(name="connectivity_monitor")
    client.hook_agent(connectivity_monitor)
    
    # Clean up resources when done
    await client.shutdown()

if __name__ == "__main__":
    asyncio.run(main())
```

## Executing Prompts

Execute prompts with automatic orchestration:

```python
import asyncio
from oblix import OblixClient, ModelType
from oblix.agents.resource_monitor import ResourceMonitor
from oblix.agents.connectivity import ConnectivityAgent

async def main():
    client = OblixClient(oblix_api_key="your_oblix_api_key")
    
    # Hook models
    await client.hook_model(ModelType.OLLAMA, "llama2")
    await client.hook_model(
        ModelType.OPENAI, 
        "gpt-3.5-turbo", 
        api_key="your_openai_api_key"
    )
    
    # Add monitoring agents
    client.hook_agent(ResourceMonitor())
    client.hook_agent(ConnectivityAgent())
    
    # Execute a prompt
    response = await client.execute("Explain quantum computing in simple terms")
    
    # Print the response
    print(response["response"])
    
    # Print which model was used
    print(f"Used model: {response['model_id']}")

if __name__ == "__main__":
    asyncio.run(main())
```

## Model Orchestration

Oblix's core strength is in orchestration between local and cloud models:

```python
# Let Oblix handle model selection based on system conditions
response = await client.execute(
    "Explain quantum computing in simple terms"
)

# Learn which model was selected
print(f"Used model: {response['model_id']}")

# You can also specify a particular model when needed
response = await client.execute(
    "Explain quantum computing in simple terms",
    model_id="openai:gpt-3.5-turbo"  # Use a specific model
)
```

> **Note:** For Oblix orchestration to work properly, ensure you've hooked at least one local model (Ollama), one cloud model (OpenAI/Claude), and the appropriate monitoring agents. You can still specify particular models when needed using the `model_id` parameter.

## Customizing Generation Parameters

Customize parameters for text generation:

```python
# Execute with custom parameters
response = await client.execute(
    "Write a short story about a robot",
    temperature=0.8,  # Controls creativity (0.0-1.0)
    max_tokens=500    # Limits response length
)
```

## Working with Sessions

Create and use sessions for conversational interactions:

```python
import asyncio
from oblix import OblixClient, ModelType

async def main():
    client = OblixClient(oblix_api_key="your_oblix_api_key")
    
    # Hook models
    await client.hook_model(ModelType.OLLAMA, "llama2")
    await client.hook_model(
        ModelType.OPENAI, 
        "gpt-3.5-turbo", 
        api_key="your_openai_api_key"
    )
    
    # Create a new session
    session_id = await client.create_session(title="My Conversation")
    
    # Set as current session
    client.current_session_id = session_id
    
    # Send messages in the session
    response1 = await client.execute("Hello, how are you today?")
    print("Assistant:", response1["response"])
    
    response2 = await client.execute("Tell me about yourself")
    print("Assistant:", response2["response"])
    
    # List all sessions
    sessions = client.list_sessions()
    print(f"Number of sessions: {len(sessions)}")

if __name__ == "__main__":
    asyncio.run(main())
```

## Simplified Chat Interface

Use the simplified chat interface for conversational applications:

```python
import asyncio
from oblix import OblixClient, ModelType

async def main():
    client = OblixClient(oblix_api_key="your_oblix_api_key")
    
    # Hook models
    await client.hook_model(ModelType.OLLAMA, "llama2")
    await client.hook_model(
        ModelType.OPENAI, 
        "gpt-3.5-turbo", 
        api_key="your_openai_api_key"
    )
    
    # Send a chat message (creates a new session)
    response = await client.chat_once("Hello, how are you today?")
    session_id = response["session_id"]
    print("Assistant:", response["response"])
    
    # Continue the conversation in the same session
    response = await client.chat_once("What's the weather like?", session_id)
    print("Assistant:", response["response"])

if __name__ == "__main__":
    asyncio.run(main())
```

## Interactive Terminal Chat

Start an interactive chat session in the terminal:

```python
import asyncio
import os
from dotenv import load_dotenv
from oblix import OblixClient, ModelType
from oblix.agents import ResourceMonitor, ConnectivityAgent

async def main():
    # Load environment variables from .env file
    load_dotenv()
    
    oblix_api_key = os.getenv('OBLIX_API_KEY')
    client = OblixClient(oblix_api_key=oblix_api_key)
    
    # Hook models
    await client.hook_model(
        ModelType.OLLAMA, 
        "llama2",
        endpoint="http://localhost:11434"
    )
    
    openai_api_key = os.getenv('OPENAI_API_KEY')
    await client.hook_model(
        ModelType.OPENAI, 
        "gpt-3.5-turbo", 
        api_key=openai_api_key
    )
    
    # Add monitoring agents
    resource_monitor = ResourceMonitor(name="resource_monitor")
    connectivity_monitor = ConnectivityAgent(name="connectivity_monitor")
    client.hook_agent(resource_monitor)
    client.hook_agent(connectivity_monitor)
    
    # Start interactive streaming chat session
    print("Starting chat session. Type 'exit' to quit.")
    print("\nSpecial commands:")
    print("  /metrics - Show current system metrics")
    print("  /status - Show orchestration status")
    print("  /local - Force next response to use local model")
    print("  /cloud - Force next response to use cloud model")
    
    await client.chat_streaming()
    
    # Clean up resources
    await client.shutdown()

if __name__ == "__main__":
    try:
        asyncio.run(main())
    except KeyboardInterrupt:
        print("\nProgram interrupted by user. Exiting...")
    except Exception as e:
        print(f"\nError: {e}")
```

## Error Handling

Handle potential errors gracefully:

```python
import asyncio
from oblix import OblixClient, ModelType

async def main():
    try:
        client = OblixClient(oblix_api_key="your_oblix_api_key")
        
        # Hook models
        await client.hook_model(ModelType.OLLAMA, "llama2")
        
        # Execute a prompt
        response = await client.execute("Explain quantum computing")
        print(response["response"])
        
    except Exception as e:
        print(f"Error: {e}")
        
    finally:
        # Clean up resources
        if 'client' in locals():
            await client.shutdown()

if __name__ == "__main__":
    asyncio.run(main())
```

## Complete Example

Here's a complete example combining multiple features:

```python
import asyncio
from oblix import OblixClient, ModelType
from oblix.agents.resource_monitor import ResourceMonitor
from oblix.agents.connectivity import ConnectivityAgent

async def main():
    try:
        # Initialize client
        client = OblixClient(oblix_api_key="your_oblix_api_key")
        
        # Hook models
        await client.hook_model(ModelType.OLLAMA, "llama2")
        await client.hook_model(
            ModelType.OPENAI, 
            "gpt-3.5-turbo", 
            api_key="your_openai_api_key"
        )
        
        # Add monitoring agents
        client.hook_agent(ResourceMonitor())
        client.hook_agent(ConnectivityAgent())
        
        # Create a session
        session_id = await client.create_session(title="AI Assistant Chat")
        client.current_session_id = session_id
        
        # Execute prompts in the session
        print("Asking about quantum computing...")
        response1 = await client.execute("Explain quantum computing briefly")
        print(f"Response from {response1['model_id']}:")
        print(response1["response"])
        print("\n" + "-"*50 + "\n")
        
        print("Asking a follow-up question...")
        response2 = await client.execute("What are some practical applications?")
        print(f"Response from {response2['model_id']}:")
        print(response2["response"])
        
        # Print agent check results
        print("\nAgent check results:")
        for agent_name, check_result in response2["agent_checks"].items():
            print(f"- {agent_name}: {check_result.get('state')}, target: {check_result.get('target')}")
        
    except Exception as e:
        print(f"Error: {e}")
        
    finally:
        # Clean up resources
        if 'client' in locals():
            await client.shutdown()

if __name__ == "__main__":
    asyncio.run(main())
```

This example demonstrates:
- Initializing the client
- Hooking multiple models
- Adding monitoring agents
- Creating and using a session
- Executing multiple prompts
- Accessing model information and agent checks
- Proper error handling and resource cleanup

## macOS-Specific Considerations

Since Oblix is currently only available for macOS, here are some platform-specific notes:

- Oblix leverages macOS-specific optimizations for resource monitoring
- For Apple Silicon Macs, Oblix can detect and utilize Metal-compatible GPUs
- The built-in agents are tuned for optimal performance on macOS
- All examples in this documentation will work on macOS 10.15 (Catalina) or newer

For more advanced examples, see [Hybrid Execution](hybrid-execution.md) and specific use cases in the examples section.

Need help? Join our community on Discord: [https://discord.gg/v8qtEVuU](https://discord.gg/v8qtEVuU)