# Agents

Agents in Oblix are monitoring components that provide system awareness and make intelligent orchestration decisions. They analyze system resources, network connectivity, and other factors to help the SDK choose the optimal model for execution.

## Agent Lifecycle

Each agent in the Oblix system follows this lifecycle:

1. **Initialization**: Configure the agent with thresholds and settings
2. **Registration**: Hook the agent to the OblixClient
3. **Monitoring**: Ongoing collection of metrics
4. **Policy Evaluation**: Assessment of metrics against policies
5. **Recommendation**: Providing execution target recommendations

## Available Agents

Oblix provides two main types of monitoring agents for orchestration:

### ResourceMonitor

Monitors system resources including CPU, memory, and GPU utilization to determine if local model execution is feasible. On macOS, it can now collect detailed GPU metrics including real-time utilization and memory usage without requiring sudo privileges.

```python
from oblix.agents import ResourceMonitor

# Create with default settings
resource_monitor = ResourceMonitor()

# Create with custom thresholds
resource_monitor = ResourceMonitor(
    name="custom_resource_monitor",
    custom_thresholds={
        "cpu_threshold": 70.0,     # CPU usage percentage (default: 80.0)
        "memory_threshold": 75.0,  # RAM usage percentage (default: 85.0)
        "load_threshold": 3.0,     # System load average (default: 4.0)
        "gpu_threshold": 75.0,     # GPU utilization percentage (default: 85.0)
        "critical_gpu": 90.0       # Critical GPU threshold (default: 95.0)
    }
)
```

#### Resource States

The resource monitor reports one of the following states:

| State | Description |
|-------|-------------|
| `AVAILABLE` | Sufficient resources available for local execution |
| `CONSTRAINED` | Resources are limited but usable |
| `CRITICAL` | Resources are extremely limited, recommend cloud execution |

#### Execution Targets

Based on resource state, the monitor recommends one of these execution targets:

| Target | Description |
|--------|-------------|
| `LOCAL_CPU` | Execute on local CPU |
| `LOCAL_GPU` | Execute on local GPU (if available and supported) |
| `CLOUD` | Execute on cloud model |

### ConnectivityAgent

Monitors network connectivity, including latency, packet loss, and bandwidth to determine if cloud model execution is viable.

```python
from oblix.agents import ConnectivityAgent

# Create with default settings
connectivity_agent = ConnectivityAgent()

# Create with custom configuration
connectivity_agent = ConnectivityAgent(
    name="custom_connectivity_monitor",
    latency_threshold=150.0,       # Maximum acceptable latency in ms (default: 200.0)
    packet_loss_threshold=5.0,     # Maximum acceptable packet loss percentage (default: 10.0)
    bandwidth_threshold=10.0,      # Minimum acceptable bandwidth in Mbps (default: 5.0)
    check_interval=60              # Seconds between connectivity checks (default: 30)
)
```

#### Connectivity States

The connectivity agent reports one of the following states:

| State | Description |
|-------|-------------|
| `OPTIMAL` | Good connectivity suitable for cloud model use |
| `DEGRADED` | Limited connectivity that may affect performance |
| `DISCONNECTED` | No connectivity, must use local models |

#### Connection Targets

Based on connectivity state, the agent recommends one of these targets:

| Target | Description |
|--------|-------------|
| `LOCAL` | Use local models due to connectivity issues |
| `CLOUD` | Use cloud models due to good connectivity |
| `HYBRID` | Consider balancing between local and cloud models |

## Orchestration Process

When working together, agents form a comprehensive orchestration system:

1. **Resource Assessment**: The ResourceMonitor assesses whether local execution is viable
2. **Connectivity Assessment**: The ConnectivityAgent determines if cloud models are accessible
3. **Conflict Resolution**: When recommendations conflict, Oblix uses these priority rules:
   - If resources are critically constrained but connectivity is good, prioritize resource constraints
   - If connectivity is poor but resources are available, prioritize connectivity constraints
   - In balanced scenarios, use policy-defined preferences

## Using Agents

Agents are added to the Oblix client with the `hook_agent()` method:

```python
from oblix import OblixClient
from oblix.agents import ResourceMonitor, ConnectivityAgent

# Initialize client
client = OblixClient(oblix_api_key="your_api_key")

# Add monitoring agents
client.hook_agent(ResourceMonitor())
client.hook_agent(ConnectivityAgent())
```

Once agents are hooked, the Oblix client automatically consults them during execution to make intelligent routing decisions.

## Agent Checks

When you execute a prompt using the `execute()` method, Oblix runs checks with all registered agents. The results of these checks are included in the response under the `agent_checks` key:

```python
response = await client.execute("Explain quantum computing")

# Access agent check results
agent_checks = response["agent_checks"]
```

The `agent_checks` dictionary contains information about:

- Current resource state
- Connectivity state
- Recommended execution target
- Detailed metrics
- Reasoning for the recommendation

## Accessing Agent Metrics Directly

You can also access the current metrics from agents directly:

```python
# Get resource metrics
resource_metrics = await client.get_resource_metrics()
print(f"CPU usage: {resource_metrics['cpu_percent']}%")

# Access GPU metrics on macOS
if resource_metrics.get('gpu') and resource_metrics['gpu'].get('available'):
    gpu_info = resource_metrics['gpu']
    if gpu_info.get('utilization') is not None:
        print(f"GPU utilization: {gpu_info['utilization'] * 100:.2f}%")
    if gpu_info.get('memory_utilization') is not None:
        print(f"GPU memory utilization: {gpu_info['memory_utilization'] * 100:.2f}%")
    print(f"GPU name: {gpu_info.get('name', 'Unknown')}")

# Get connectivity metrics
connectivity_metrics = await client.get_connectivity_metrics()
print(f"Current latency: {connectivity_metrics['latency']}ms")
```

This direct access is useful for monitoring and debugging orchestration behavior.