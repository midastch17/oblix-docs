# Agents

Agents are a core component of the Oblix orchestration system that provide awareness of system state and influence model orchestration decisions. This page explains how agents work and how they are used to optimize AI model execution.

## What Are Agents?

In Oblix, agents are specialized components that:

1. **Monitor** specific aspects of the system (resources, connectivity, etc.)
2. **Evaluate** the current state against defined thresholds
3. **Recommend** execution targets based on their findings
4. **Provide** detailed metrics for transparency and debugging

Agents run continuously in the background, ensuring the Oblix system always has up-to-date information about the execution environment to make intelligent orchestration decisions.

## Built-in Agent Types

Oblix currently provides two built-in agent types that work together to make intelligent orchestration decisions:

### ResourceMonitor

The `ResourceMonitor` agent tracks system resource utilization, including:

- **CPU usage** - Overall processor utilization percentage
- **Memory usage** - RAM utilization percentage
- **System load** - Average system load over time
- **GPU availability** - Presence and utilization of GPU resources
- **GPU metrics** - Real-time GPU utilization and memory usage on macOS devices

Based on these metrics, the ResourceMonitor recommends one of several execution targets:

- `LOCAL_CPU` - Execute on local CPU (when resources are available)
- `LOCAL_GPU` - Execute on local GPU (when available and suitable)
- `CLOUD` - Execute on cloud models (when local resources are constrained)

```python
from oblix.agents.resource_monitor import ResourceMonitor

# Create with default settings
resource_monitor = ResourceMonitor()

# Add to client
client.hook_agent(resource_monitor)
```

#### Resource States

The ResourceMonitor reports one of three states:

- `AVAILABLE` - Sufficient resources available for local execution
- `CONSTRAINED` - Resources are limited but usable
- `CRITICAL` - Resources are extremely limited, recommend cloud execution

### ConnectivityAgent

The `ConnectivityAgent` monitors network connectivity, including:

- **Connection type** - Type of network connection (wifi, ethernet, etc.)
- **Latency** - Response time to key endpoints
- **Packet loss** - Percentage of lost packets
- **Bandwidth** - Available network bandwidth

Based on these metrics, the ConnectivityAgent recommends one of several orchestration targets:

- `LOCAL` - Use local models due to connectivity issues
- `CLOUD` - Use cloud models due to good connectivity
- `HYBRID` - Consider balanced execution between local and cloud

```python
from oblix.agents.connectivity import ConnectivityAgent

# Create with default settings
connectivity_agent = ConnectivityAgent()

# Add to client
client.hook_agent(connectivity_agent)
```

#### Connectivity States

The ConnectivityAgent reports one of three states:

- `OPTIMAL` - Good connectivity suitable for cloud model use
- `DEGRADED` - Limited connectivity that may affect performance
- `DISCONNECTED` - No connectivity, must use local models

## Agent Lifecycle

Agents follow a defined lifecycle:

1. **Initialization** - Setting up monitoring capabilities
2. **Periodic Checks** - Collecting metrics at regular intervals
3. **On-demand Checks** - Performing checks when explicitly requested
4. **Shutdown** - Gracefully releasing resources

## Agent Orchestration Process

When you execute a prompt, all registered agents perform checks to inform the orchestration decision:

```python
response = await client.execute("Explain quantum computing")

# Orchestration decisions are included in the response
routing_decision = response["routing_decision"]  # Overall orchestration decisions
agent_checks = response["agent_checks"]          # Detailed agent results
```

The `routing_decision` dictionary contains high-level orchestration decisions, while `agent_checks` contains detailed information from each agent:

- Current system state assessments
- Recommended execution targets
- Specific metrics collected
- Reasoning for orchestration decisions

Note that `agent_checks` is a list of dictionaries, with each dictionary containing:
```python
{
    "agent": "resource_monitor",     # Name of the agent
    "result": {                      # Agent check results
        "proceed": True,             # Whether the execution should proceed
        "state": "available",        # Current state detected
        "target": "local_gpu",       # Recommended target
        "reason": "Resources available",  # Reasoning
        "metrics": { ... }           # Detailed metrics
    }
}
```

## Customizing Agent Behavior

You can customize agent behavior by providing configuration options:

### Resource Thresholds

```python
# Create with custom thresholds
resource_monitor = ResourceMonitor(
    custom_thresholds={
        "cpu_threshold": 70.0,     # CPU usage percentage (default: 80.0)
        "memory_threshold": 75.0,  # RAM usage percentage (default: 85.0)
        "load_threshold": 3.0,     # System load average (default: 4.0)
        "gpu_threshold": 75.0,     # GPU utilization percentage threshold (default: 85.0)
        "critical_gpu": 90.0       # Critical GPU threshold (default: 95.0)
    }
)
```

### Connectivity Thresholds

```python
# Create with custom connectivity thresholds
connectivity_agent = ConnectivityAgent(
    latency_threshold=150.0,       # Maximum acceptable latency in ms (default: 200.0)
    packet_loss_threshold=5.0,     # Maximum acceptable packet loss percentage (default: 10.0)
    bandwidth_threshold=10.0,      # Minimum acceptable bandwidth in Mbps (default: 5.0)
    check_interval=60              # Seconds between connectivity checks (default: 30)
)
```

## Intelligent Orchestration System

Oblix's agent architecture powers its intelligent orchestration system with several key benefits:

1. **Adaptive Execution** - Dynamically selects the optimal model based on real-time conditions
2. **Resilience** - Automatically falls back to local models when connectivity fails
3. **Resource Optimization** - Balances workload between local and cloud resources
4. **Performance Tuning** - Configurable thresholds to match specific hardware capabilities
5. **Transparency** - Clear visibility into orchestration decisions

## Orchestration Decision Flow

The Oblix orchestration system follows this decision process:

1. **Resource Assessment** - ResourceMonitor evaluates system capabilities
2. **Connectivity Verification** - ConnectivityAgent checks network conditions
3. **Decision Prioritization** - Weighs connectivity status higher than resource availability
4. **Model Selection** - Chooses appropriate model based on agent recommendations
5. **Execution** - Runs the prompt on the selected model

This orchestration flow ensures that your AI workloads run on the most appropriate model given the current system conditions.

## Extensible Agent System

Oblix's agent system is designed to be extensible and scalable. The architecture allows for additional agents to be integrated in the future, enhancing the orchestration capabilities while maintaining compatibility with existing code. As new monitoring needs emerge, the agent ecosystem can grow accordingly.

## GPU Monitoring on macOS

Oblix provides enhanced GPU monitoring capabilities on macOS devices:

- **Real-time GPU Utilization** - Live measurement of GPU processing load (0-100%)
- **Memory Usage Tracking** - For Apple Silicon GPUs with unified memory
- **No Sudo Required** - Works without elevated privileges (permissions handled during installation)
- **Automatic Decision Making** - Routes to appropriate execution target based on GPU state

### Accessing GPU Metrics

You can access GPU metrics programmatically:

```python
# Get complete resource metrics
resource_metrics = await client.get_resource_metrics()

# Access GPU metrics
if resource_metrics.get('gpu') and resource_metrics['gpu'].get('available'):
    gpu_info = resource_metrics['gpu']
    
    # Check GPU utilization
    if gpu_info.get('utilization') is not None:
        print(f"GPU utilization: {gpu_info['utilization'] * 100:.2f}%")
    
    # Memory usage (Apple Silicon)
    if gpu_info.get('memory_utilization') is not None:
        print(f"GPU memory: {gpu_info['memory_utilization'] * 100:.2f}%")
    
    # Additional information
    print(f"GPU name: {gpu_info.get('name', 'Unknown')}")
    print(f"GPU type: {gpu_info.get('type', 'Unknown')}")
```

## Best Practices

When working with Oblix's agent-based orchestration:

- **Enable Both Agents** - Use both ResourceMonitor and ConnectivityAgent together for optimal orchestration
- **Start with Defaults** - Use the default agent thresholds initially
- **Monitor Performance** - Review agent recommendations over time
- **Adjust Gradually** - Make small adjustments to thresholds based on observations
- **Configure for Your Environment** - Tune thresholds based on your specific hardware capabilities
- **Tune GPU Thresholds** - Adjust GPU thresholds based on your specific GPU model and workload requirements

By leveraging Oblix's agent-based orchestration system effectively, you can build AI applications that intelligently adapt to changing conditions while optimizing for performance, cost, and reliability.