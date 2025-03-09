# How Oblix Works

Oblix is an AI model orchestration SDK that intelligently routes between local and cloud-based models based on connectivity, system resources, and other factors. This page explains the core architecture and concepts that power Oblix.

## Core Architecture

![Oblix Architecture Diagram](assets/oblix-architecture.png)

Oblix consists of several key components:

1. **Client Interface** - The main entry point (`OblixClient`) that developers interact with
2. **Model Management** - Handles various model types from different providers
3. **Agent System** - Monitors system state and makes routing decisions
4. **Session Management** - Maintains persistent chat sessions
5. **Execution Engine** - Orchestrates the execution flow between models

## Intelligent Routing

Oblix's intelligent routing system works by:

1. **Monitoring Resources** - Tracking CPU, memory, and GPU utilization
2. **Checking Connectivity** - Monitoring network quality and availability  
3. **Applying Policies** - Using configurable policies to make routing decisions
4. **Executing Actions** - Dynamically switching between models based on current conditions

### Example Routing Scenarios

| Scenario | Resource State | Connectivity | Routing Decision |
|----------|---------------|--------------|------------------|
| Offline work | Available | Disconnected | Route to local Ollama model |
| Limited bandwidth | Available | Degraded | Use smaller cloud model or local model |
| Resource constrained | Constrained | Optimal | Route to cloud model |
| Optimal conditions | Available | Optimal | Prefer local model for speed |

## Agent System

Agents are pluggable components that monitor specific aspects of the system:

### ResourceMonitor

Tracks system resources including:
- CPU utilization and load
- Memory availability
- GPU availability and utilization

Based on resource metrics, it recommends the optimal execution target (local CPU, local GPU, or cloud).

### ConnectivityAgent

Monitors network connectivity including:
- Connection type (wifi, ethernet, etc.)
- Latency and packet loss
- Available bandwidth
- Connection stability

The connectivity agent adapts to changing network conditions, ensuring reliability even in challenging environments.

## Model Abstraction

Oblix provides a unified interface for working with different model types:

- **Ollama Models** - Local models running via Ollama
- **OpenAI Models** - Cloud models from OpenAI (GPT series)
- **Claude Models** - Cloud models from Anthropic (Claude series)

Each model implementation handles:
- Authentication and API communication
- Token counting and management
- Performance metrics collection
- Error handling and retries

## Session Management

Oblix includes a built-in session management system that:

- Maintains conversation history
- Provides persistence across application restarts
- Enables multi-session workflows
- Handles context management for stateful conversations

## Execution Flow

When you execute a prompt with Oblix, here's what happens behind the scenes:

1. **Pre-execution Checks** - All registered agents perform checks and make recommendations
2. **Model Selection** - The system selects the optimal model based on agent recommendations
3. **Context Retrieval** - If in a session, relevant context is loaded
4. **Execution** - The prompt is sent to the selected model for processing
5. **Metrics Collection** - Performance metrics are collected during execution
6. **Response Handling** - The response is returned along with metadata
7. **Session Update** - If in a session, the interaction is saved

## Configuration System

Oblix stores configuration persistently, including:

- Model configurations
- API keys (securely)
- Custom thresholds
- User preferences

The configuration system provides portability and simplifies repeated use.

## Optimizing for Developer Experience

Oblix abstracts away the complexity of working with multiple models, allowing developers to:

- Write code once that works across changing environments
- Gracefully handle offline scenarios
- Optimize for performance and cost
- Maintain conversation context seamlessly

This design ensures that AI applications built with Oblix are more resilient, adaptive, and user-friendly than traditional approaches.
