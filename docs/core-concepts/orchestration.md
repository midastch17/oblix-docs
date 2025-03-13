Here's the complete content for your `core-concepts/orchestration.md` file:

# Orchestration

Orchestration is the heart of the Oblix SDK, providing intelligent routing between local and cloud models based on system state. This page explains how Oblix's orchestration system works.

## The Orchestration Challenge

AI applications face several challenges when deciding which models to use:

- **Resource constraints**: Local devices may have limited CPU, memory, or GPU resources
- **Connectivity issues**: Network quality can vary or be completely unavailable
- **Cost considerations**: Cloud models incur API costs per token
- **Performance requirements**: Response time may be critical for certain applications
- **Capability needs**: Some tasks require more advanced models

Traditionally, developers had to manually implement logic to handle these scenarios, leading to complex conditional code or simplified approaches that don't adapt to changing conditions.

## Oblix's Orchestration Solution

Oblix solves this challenge with a dynamic orchestration system that:

1. **Continuously monitors** system resources and connectivity
2. **Applies configurable policies** to determine the optimal execution target
3. **Automatically routes** prompts to the best available model
4. **Provides transparency** into routing decisions

## Orchestration Components

### Monitoring Agents

Agents collect real-time metrics about the system state:

- **ResourceMonitor**: Collects CPU, memory, and GPU utilization metrics
- **ConnectivityAgent**: Measures latency, packet loss, and bandwidth

### Policy Evaluation

Policies define the rules for routing decisions:

- **ResourcePolicy**: Evaluates system resource metrics against thresholds
- **ConnectivityPolicy**: Evaluates network quality against thresholds

### Routing Decision

Based on policy evaluation, the orchestration system determines:

- Which model provider to use (Ollama, OpenAI, Claude)
- Which specific model to execute the prompt with
- What parameters to apply for optimal performance

## Orchestration Flow

When you execute a prompt, the orchestration process follows these steps:

1. **Agent Checks**: All registered agents perform checks and report their findings
2. **Policy Evaluation**: Resource and connectivity policies evaluate the current state
3. **Target Selection**: The system selects the optimal execution target
4. **Model Selection**: The appropriate model is chosen from the target provider
5. **Execution**: The prompt is sent to the selected model
6. **Response Handling**: The response is returned along with execution metadata

## Orchestration States

The orchestration system can be in various states that influence routing decisions:

### Resource States

| State | Description | Typical Routing |
|-------|-------------|-----------------|
| **AVAILABLE** | Sufficient resources available | Local execution preferred |
| **CONSTRAINED** | Limited resources available | Smaller local models or cloud |
| **CRITICAL** | Extremely limited resources | Cloud execution required |

### Connectivity States

| State | Description | Typical Routing |
|-------|-------------|-----------------|
| **OPTIMAL** | Good connectivity | Cloud execution viable |
| **DEGRADED** | Limited connectivity | Smaller cloud models or local |
| **DISCONNECTED** | No connectivity | Local execution required |

## Configuring Orchestration

You can customize the orchestration system by:

### Custom Resource Thresholds

```python
from oblix.agents import ResourceMonitor

# Create with custom thresholds
resource_monitor = ResourceMonitor(
    custom_thresholds={
        "cpu_threshold": 70.0,     # CPU usage percentage (default: 80.0)
        "memory_threshold": 75.0,  # RAM usage percentage (default: 85.0)
        "load_threshold": 3.0,     # System load average (default: 4.0)
        "gpu_threshold": 75.0,     # GPU utilization percentage (default: 85.0)
        "critical_gpu": 90.0       # Critical GPU threshold (default: 95.0)
    }
)
```

### Custom Connectivity Thresholds

```python
from oblix.agents import ConnectivityAgent

# Create with custom connectivity thresholds
connectivity_agent = ConnectivityAgent(
    latency_threshold=150.0,       # Maximum acceptable latency in ms (default: 200.0)
    packet_loss_threshold=5.0,     # Maximum acceptable packet loss percentage (default: 10.0)
    bandwidth_threshold=10.0       # Minimum acceptable bandwidth in Mbps (default: 5.0)
)
```

## Transparency and Debugging

Oblix provides transparency into orchestration decisions through the response metadata:

```python
response = await client.execute("Explain quantum computing")

# Access orchestration decision data
agent_checks = response["agent_checks"]
print(f"Resource state: {agent_checks.get('resource_monitor', {}).get('state')}")
print(f"Connectivity: {agent_checks.get('connectivity_monitor', {}).get('state')}")
print(f"Selected model: {response['model_id']}")
```

This transparency helps you understand why specific routing decisions were made and can be valuable for debugging or fine-tuning the orchestration system.

## Fallback Mechanisms

If a preferred model is unavailable or fails, Oblix includes fallback mechanisms:

1. **Primary execution attempt** with the optimal model
2. **Fallback to alternative models** if the primary fails
3. **Degraded mode operation** when connectivity is limited

This ensures robustness even in challenging environments.

## Advanced Orchestration Features

### Model-Specific Routing

You can override automatic routing for specific prompts:

```python
# Force execution with a specific model
response = await client.execute(
    "Explain quantum computing",
    model_id="openai:gpt-4"
)
```

### Session-Based Routing

For chat sessions, Oblix maintains routing consistency when possible, to provide a seamless experience:

```python
# Start a chat session (routing decisions persist across messages)
await client.chat_streaming()
```

### Multi-Model Execution

For critical applications, you can implement multi-model execution strategies:

```python
# Execute with multiple models and compare results
response1 = await client.execute("Explain quantum computing", model_id="openai:gpt-4")
response2 = await client.execute("Explain quantum computing", model_id="claude:claude-3-opus-20240229")
```

By understanding and configuring Oblix's orchestration system, you can build AI applications that seamlessly adapt to changing conditions while optimizing for cost, performance, and reliability.
