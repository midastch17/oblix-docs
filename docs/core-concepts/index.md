# Core Concepts

This section explains the fundamental concepts behind the Oblix SDK and how it works to provide intelligent AI model orchestration between edge and cloud environments.

## What is Oblix Orchestration?

Oblix orchestration is the process of intelligently managing and coordinating multiple AI models between edge (local) and cloud environments to optimize for factors such as:

- Performance and latency
- Resource utilization
- Cost efficiency
- Reliability
- Connectivity status

Rather than manually deciding which model to use in different situations, Oblix automatically orchestrates this decision-making process based on real-time system metrics and connectivity data, providing seamless switching between environments.

## How Oblix Works

![Oblix Architecture Diagram](../assets/oblix-architecture.png)

At its core, Oblix works by:

1. **Providing a unified interface** for multiple model providers (OpenAI, Claude, Ollama)
2. **Monitoring system metrics** through specialized agents
3. **Evaluating orchestration policies** based on collected metrics
4. **Intelligently orchestrating executions** between edge and cloud models
5. **Managing session context** for conversational interactions

## Key Components

### OblixClient

The main entry point for the SDK provides methods for:

- Hooking models from different providers
- Adding monitoring agents for orchestration
- Executing prompts with seamless switching
- Managing chat sessions

### Models

Oblix supports multiple model types:

- **OLLAMA**: Edge models running locally via Ollama
- **OPENAI**: Cloud models from OpenAI (GPT series)
- **CLAUDE**: Cloud models from Anthropic (Claude series)
- **CUSTOM**: Custom model implementations

### Agents

Specialized components that monitor system state and enable intelligent orchestration:

- **ResourceMonitor**: Tracks CPU, memory, and GPU usage for edge execution decisions
- **ConnectivityAgent**: Monitors network connectivity quality for cloud execution decisions

### Orchestration Flow

When you execute a prompt with Oblix:

1. All registered agents perform checks on system resources and connectivity
2. Orchestration policies evaluate the collected metrics
3. The optimal model is selected based on the current state
4. The prompt is sent to the selected model
5. The response is returned along with execution metadata

### Edge-Cloud Execution Strategy

Oblix implements an intelligent edge-cloud execution strategy that seamlessly switches between:

- **Edge execution** for better privacy, lower latency, and offline capability
- **Cloud execution** for more powerful models and when local resources are constrained

## Benefits

This orchestration approach provides several key benefits:

- **Resilience**: Continue operating even when connectivity is limited or lost
- **Optimization**: Use the most cost-effective model for each task
- **Performance**: Select models based on current system capabilities
- **Simplicity**: Provide a single interface for all model interactions
- **Adaptability**: Automatically adjust to changing conditions

In the following sections, we'll dive deeper into each of these core concepts:

- [Orchestration](orchestration.md): How the orchestration system works
- [Agents](agents.md): Monitoring system state for orchestration decisions
- [Session Management](session-management.md): Handling conversations

Understanding these concepts will help you get the most out of the Oblix SDK in your applications.