# Basic Usage

This page provides basic examples to help you get started with Oblix. These examples cover installation, initialization, model hooking, and executing prompts with the SDK's orchestration capabilities.

## Installation

Oblix is available exclusively for macOS at this time:

```bash
pip install oblix
```

> **Note:** Currently, only macOS is supported. Windows and Linux versions are planned for future releases. Oblix supports Python versions 3.9, 3.10, 3.11, 3.12, and 3.13.

## Prerequisites

Before using Oblix with edge models, you'll need to:

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

## Hooking Models for Orchestration

Add both edge and cloud models to enable orchestration:

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
    
    # Hook an edge model via Ollama (replace with any model you've pulled)
    await client.hook_model(
        model_type=ModelType.OLLAMA,
        model_name="llama2",  # Use any available Ollama model
        endpoint="http://localhost:11434"  # Default Ollama endpoint
    )
    
    # Hook a cloud model via OpenAI
    openai_api_key = os.getenv('OPENAI_API_KEY')
    await client.hook_model(
        model_type=ModelType.OPENAI,
        model_name="gpt-3.5-turbo",
        api_key=openai_api_key
    )
    
    # Optionally hook another cloud model via Claude
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

## Adding Monitoring Agents for Orchestration

Add monitoring agents to enable intelligent orchestration between edge and cloud:

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
    
    # Hook edge and cloud models for orchestration
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
    
    # Add resource monitoring for edge execution decisions
    resource_monitor = ResourceMonitor(name="resource_monitor")
    client.hook_agent(resource_monitor)
    
    # Add connectivity monitoring for cloud execution decisions
    connectivity_monitor = ConnectivityAgent(name="connectivity_monitor")
    client.hook_agent(connectivity_monitor)
    
    # Clean up resources when done
    await client.shutdown()

if __name__ == "__main__":
    asyncio.run(main())
```

## Executing Prompts with Orchestration

Execute prompts with automatic orchestration between edge and cloud:

```python
import asyncio
from oblix import OblixClient, ModelType
from oblix.agents.resource_monitor import ResourceMonitor
from oblix.agents.connectivity import ConnectivityAgent

async def main():
    client = OblixClient(oblix_api_key="your_oblix_api_key")
    
    # Hook edge and cloud models for orchestration
    await client.hook_model(ModelType.OLLAMA, "llama2")
    await client.hook_model(
        ModelType.OPENAI, 
        "gpt-3.5-turbo", 
        api_key="your_openai_api_key"
    )
    
    # Add monitoring agents for orchestration
    client.hook_agent(ResourceMonitor())
    client.hook_agent(ConnectivityAgent())
    
    # Execute a prompt with intelligent orchestration
    response = await client.execute("Explain quantum computing in simple terms")
    
    # Print the response
    print(response["response"])
    
    # Print which model was used by the orchestration
    print(f"Model used by orchestration: {response['model_id']}")

if __name__ == "__main__":
    asyncio.run(main())
```

## Edge-Cloud Orchestration

Oblix's core strength is in orchestration between edge and cloud models:

```python
# Let Oblix handle orchestration based on system conditions
response = await client.execute(
    "Explain quantum computing in simple terms"
)

# Learn which model was selected by the orchestration
print(f"Model used by orchestration: {response['model_id']}")

# You can also override orchestration when needed
response = await client.execute(
    "Explain quantum computing in simple terms",
    model_id="openai:gpt-3.5-turbo"  # Explicitly use cloud model
)
```

> **Note:** For Oblix orchestration to work properly, ensure you've hooked at least one edge model (Ollama), one cloud model (OpenAI/Claude), and the appropriate monitoring agents. You can still override orchestration when needed using the `model_id` parameter.

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

## Streaming Generation

For real-time streaming of tokens as they're generated:

```python
import asyncio
from oblix import OblixClient, ModelType
from oblix.agents import ResourceMonitor, ConnectivityAgent

async def main():
    client = OblixClient(oblix_api_key="your_oblix_api_key")
    
    # Hook models for orchestration
    await client.hook_model(ModelType.OLLAMA, "llama2")
    await client.hook_model(
        ModelType.OPENAI, 
        "gpt-3.5-turbo", 
        api_key="your_openai_api_key"
    )
    
    # Add monitoring agents
    client.hook_agent(ResourceMonitor())
    client.hook_agent(ConnectivityAgent())
    
    # Stream a response with automatic orchestration
    # Tokens are printed to the console in real-time
    response = await client.execute_streaming("Write a short poem about AI")
    
    print(f"\nModel used: {response['model_id']}")
    
    await client.shutdown()

if __name__ == "__main__":
    asyncio.run(main())
```

## Working with Sessions

Create and use sessions for conversational interactions with orchestration:

```python
import asyncio
from oblix import OblixClient, ModelType

async def main():
    client = OblixClient(oblix_api_key="your_oblix_api_key")
    
    # Hook models for orchestration
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
    
    # Send messages in the session with orchestration
    response1 = await client.execute("Hello, how are you today?")
    print(f"Assistant ({response1['model_id']}): {response1['response']}")
    
    response2 = await client.execute("Tell me about yourself")
    print(f"Assistant ({response2['model_id']}): {response2['response']}")
    
    # List all sessions
    sessions = client.list_sessions()
    print(f"Number of sessions: {len(sessions)}")

if __name__ == "__main__":
    asyncio.run(main())
```

## Simplified Chat Interface with Orchestration

Use the simplified chat interface for conversational applications:

```python
import asyncio
from oblix import OblixClient, ModelType

async def main():
    client = OblixClient(oblix_api_key="your_oblix_api_key")
    
    # Hook models for orchestration
    await client.hook_model(ModelType.OLLAMA, "llama2")
    await client.hook_model(
        ModelType.OPENAI, 
        "gpt-3.5-turbo", 
        api_key="your_openai_api_key"
    )
    
    # Send a chat message with orchestration (creates a new session)
    response = await client.chat_once("Hello, how are you today?")
    session_id = response["session_id"]
    print(f"Assistant ({response['model_id']}): {response['response']}")
    
    # Continue the conversation in the same session
    response = await client.chat_once("What's the weather like?", session_id)
    print(f"Assistant ({response['model_id']}): {response['response']}")

if __name__ == "__main__":
    asyncio.run(main())
```

## Interactive Terminal Chat with Orchestration

Start an interactive chat session in the terminal with orchestration:

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
    
    # Hook edge and cloud models for orchestration
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
    
    # Add monitoring agents for orchestration
    resource_monitor = ResourceMonitor(name="resource_monitor")
    connectivity_monitor = ConnectivityAgent(name="connectivity_monitor")
    client.hook_agent(resource_monitor)
    client.hook_agent(connectivity_monitor)
    
    # Start interactive streaming chat session with orchestration
    print("Starting chat session with edge-cloud orchestration. Type 'exit' to quit.")
    print("\nSpecial commands:")
    print("  /metrics - Show current system metrics")
    print("  /status - Show orchestration status")
    print("  /edge - Force next response to use edge model")
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
        
        # Hook edge model for orchestration
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

Here's a complete example combining multiple features with orchestration:

```python
import asyncio
from oblix import OblixClient, ModelType
from oblix.agents.resource_monitor import ResourceMonitor
from oblix.agents.connectivity import ConnectivityAgent

async def main():
    try:
        # Initialize client
        client = OblixClient(oblix_api_key="your_oblix_api_key")
        
        # Hook edge and cloud models for orchestration
        await client.hook_model(ModelType.OLLAMA, "llama2")
        await client.hook_model(
            ModelType.OPENAI, 
            "gpt-3.5-turbo", 
            api_key="your_openai_api_key"
        )
        
        # Add monitoring agents for orchestration
        client.hook_agent(ResourceMonitor())
        client.hook_agent(ConnectivityAgent())
        
        # Create a session
        session_id = await client.create_session(title="AI Assistant Chat")
        client.current_session_id = session_id
        
        # Execute prompts in the session with orchestration
        print("Asking about quantum computing...")
        response1 = await client.execute("Explain quantum computing briefly")
        print(f"Response from {response1['model_id']}:")
        print(response1["response"])
        print("\n" + "-"*50 + "\n")
        
        print("Asking a follow-up question...")
        response2 = await client.execute("What are some practical applications?")
        print(f"Response from {response2['model_id']}:")
        print(response2["response"])
        
        # Print orchestration decision information
        print("\nOrchestration decisions:")
        if "routing_decision" in response2:
            for decision_type, details in response2["routing_decision"].items():
                print(f"- {decision_type}: {details.get('state')}, target: {details.get('target')}")
        elif "agent_checks" in response2:
            # Handle both list and dict formats for backward compatibility
            if isinstance(response2["agent_checks"], dict):
                for agent_name, check_result in response2["agent_checks"].items():
                    print(f"- {agent_name}: {check_result.get('state')}, target: {check_result.get('target')}")
            elif isinstance(response2["agent_checks"], list):
                for check in response2["agent_checks"]:
                    agent_name = check.get("agent", "unknown")
                    result = check.get("result", {})
                    print(f"- {agent_name}: {result.get('state')}, target: {result.get('target')}")
        
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
- Hooking multiple models for orchestration
- Adding monitoring agents for orchestration decisions
- Creating and using a session
- Executing multiple prompts with orchestration
- Accessing model information and orchestration decisions
- Proper error handling and resource cleanup

## macOS-Specific Considerations

Since Oblix is currently only available for macOS, here are some platform-specific notes:

- Oblix leverages macOS-specific optimizations for resource monitoring
- For Apple Silicon Macs, Oblix can detect and utilize Metal-compatible GPUs
- The built-in agents are tuned for optimal orchestration on macOS
- All examples in this documentation will work on macOS 10.15 (Catalina) or newer

For more advanced examples, see [Hybrid Execution](hybrid-execution.md) and specific use cases in the examples section.

Need help? Join our community on Discord: [https://discord.gg/v8qtEVuU](https://discord.gg/v8qtEVuU)