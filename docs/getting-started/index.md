# Getting Started

Welcome to Oblix! This section will guide you through the process of installing the SDK, setting up your environment, and creating your first Oblix application.

## What is Oblix?

Oblix is a comprehensive SDK for intelligent orchestration of AI models between edge and cloud environments. The SDK provides:

- Unified interface for multiple AI model providers (OpenAI, Claude, Ollama)
- Intelligent orchestration that seamlessly switches between edge and cloud models
- System resource monitoring and connectivity awareness through specialized agents
- Persistent chat session management
- Agent system for extensible monitoring and orchestration decision making

## Installation

Oblix is currently available for macOS only and can be installed using pip:

```bash
pip install oblix
```

For a specific Python version, you can use:

```bash
python3.11 -m pip install oblix
```

Oblix supports Python versions 3.9, 3.10, 3.11, 3.12, and 3.13.

## Prerequisites

Before using Oblix, you'll need:

1. **macOS** - Oblix currently only supports macOS
2. **Oblix API Key** for authentication (see [Authentication](authentication.md))
3. **Provider API Keys** for any cloud models you want to use
4. **Ollama** installed if you want to use local edge models

## Next Steps

To continue getting started with Oblix, check out these guides:

- [Quickstart](quickstart.md) - A more comprehensive quickstart guide
- [Authentication](authentication.md) - Setting up authentication

Once you're set up, you can explore the [core concepts](../core-concepts/index.md) behind Oblix and the [API reference](../api-reference/index.md) for detailed information about the available functions and classes.