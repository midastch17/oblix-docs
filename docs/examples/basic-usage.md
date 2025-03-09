# Basic Usage

This page provides basic examples to help you get started with Oblix. These examples cover installation, initialization, model hooking, and executing prompts with the SDK.

## Installation

Oblix is available exclusively for macOS at this time:

1. Visit [oblix.ai](https://oblix.ai) to download the latest version
2. Follow the installation instructions on the website
3. Once installed, you can import the SDK in your Python projects

> **Note:** Currently, only macOS is supported. Windows and Linux versions are planned for future releases.

## Initialization

Initialize the Oblix client with your API key:

```python
import asyncio
from oblix import OblixClient

async def main():
    # Initialize with API key
    client = OblixClient(oblix_api_key="your_oblix_api_key")
    
    # Alternatively, set the OBLIX_API_KEY environment variable
    # and initialize without explicitly providing the key
    # client = OblixClient()
    
    # Rest of your code here...

# Run the async function
if __name__ == "__main__":
    asyncio.run(main())
```

## Hooking Models

Add models to your client for execution:

```python
import asyncio
from oblix import OblixClient, ModelType

async def main():
    client = OblixClient(oblix_api_key="your_oblix_api_key")
    
    # Hook a local Ollama model
    await client.hook_model(
        model_type=ModelType.OLLAMA,
        model_name="llama2"
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

if __name__ == "__main__":
    asyncio.run(main())
```

## Adding Monitoring Agents

Add monitoring agents to enable intelligent orchestration:

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
    
    # Add resource monitoring
    client.hook_agent(ResourceMonitor())
    
    # Add connectivity monitoring
    client.hook_agent(ConnectivityAgent())

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

## Using Specific Models

Force execution with a specific model:

```python
# Execute with a specific model
response = await client.execute(
    "Explain quantum computing in simple terms",
    model_id="openai:gpt-3.5-turbo"
)
```

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
    
    # Start interactive chat session
    print("Starting chat session. Type 'exit' to quit.")
    await client.start_chat()

if __name__ == "__main__":
    asyncio.run(main())
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