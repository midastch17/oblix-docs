# Agents

Agents in Oblix are monitoring components that provide system awareness and make intelligent routing decisions. They analyze system resources, network connectivity, and other factors to help the SDK choose the optimal model for execution.

## Available Agents

Oblix provides two main types of monitoring agents:

### ResourceMonitor

Monitors system resources including CPU, memory, and GPU utilization.

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
        "gpu_threshold": 75.0      # GPU utilization percentage (default: 85.0)
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

Monitors network connectivity, including latency, packet loss, and bandwidth.

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

