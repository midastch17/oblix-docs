# CLI Reference

Oblix provides a powerful command-line interface (CLI) that enables you to interact with the Oblix AI Orchestration SDK directly from your terminal. This tool simplifies hybrid model orchestration between cloud and edge environments.

## Installation

The Oblix CLI is included with the main Oblix SDK package when you install it via pip:

```bash
pip install oblix
```

> **Note:** Currently, Oblix only supports macOS. Windows and Linux versions are planned for future releases. Supported Python versions: 3.9, 3.10, 3.11, 3.12, 3.13.

## Basic Usage

Once installed, you can access the CLI by using the `oblix` command in your terminal:

```bash
oblix --help
```

This will display the available commands and options.

## Command Structure

The Oblix CLI follows this general structure:

```bash
oblix [command] [subcommand] [options]
```

## Orchestration Capabilities

Oblix is designed specifically for orchestrating AI workloads between cloud and edge environments. Rather than running models in isolation, Oblix's CLI provides:

- **Automatic orchestration** between local and cloud models based on resource availability and connectivity
- **Dynamic routing** of prompts to the most appropriate model
- **Monitoring agents** that continuously assess system state
- **Seamless fallback** between models when conditions change

## Available Commands

| Command | Description |
|---------|-------------|
| `models` | Show which AI models Oblix supports |
| `agents` | Show the monitoring agents that help Oblix make smart decisions |
| `sessions` | View and manage your chat history |
| `server` | Start the Oblix API server to use with any client application |
| `chat` | Start an interactive AI chat in your terminal |
| `check-updates` | Check if a newer version of Oblix is available |

## Authentication

Oblix no longer requires an API key for general usage. However, provider-specific API keys are still needed for cloud models:

1. For OpenAI models, set the `OPENAI_API_KEY` environment variable
2. For Claude models, set the `ANTHROPIC_API_KEY` environment variable

Example:
```bash
export OPENAI_API_KEY="your_openai_api_key"
export ANTHROPIC_API_KEY="your_anthropic_api_key"
```

## Common Options

These options are available for most commands:

| Option | Description |
|--------|-------------|
| `--help`, `-h` | Display help for a command |
| `--version` | Show the Oblix CLI version |
| `--debug` | Enable debug logging |

## Models Command

The `models` command provides information about supported model providers:

```bash
oblix models
```

This command shows all supported AI model providers that Oblix works with, including:
- Ollama: Local models served through Ollama
- OpenAI: GPT models via OpenAI API
- Claude: Claude models via Anthropic API

## Agents Command

The `agents` command shows the monitoring agents Oblix uses for orchestration:

```bash
oblix agents
```

This command displays information about:
- Resource Monitor: Monitors system resources like CPU, memory, and GPU
- Connectivity Agent: Monitors network connectivity and latency

## Sessions Commands

Commands for managing your chat history:

### List Sessions

List recent chat sessions:

```bash
oblix sessions list [--limit NUMBER]
```

### View Session

View details of a specific session:

```bash
oblix sessions view SESSION_ID
```

### Delete Session

Delete a specific chat session:

```bash
oblix sessions delete SESSION_ID
```

### Create Session

Create a new chat session:

```bash
oblix sessions create --title "My New Chat"
```

## Server Command

Start an OpenAI-compatible API server powered by Oblix orchestration:

```bash
# Start the server with specified local and cloud models
oblix server --local-model ollama:llama2 --cloud-model openai:gpt-3.5-turbo

# Start with Claude as the cloud model
oblix server --local-model ollama:llama2 --cloud-model claude:claude-3-haiku --cloud-api-key YOUR_ANTHROPIC_KEY
```

The server provides an OpenAI-compatible endpoint that can be used with any client that supports the OpenAI API.

### Server Options

| Option | Description |
|--------|-------------|
| `--port` | Port to run the server on (default: 62549) |
| `--host` | Host to bind the server to (default: 0.0.0.0) |
| `--local-model` | Local model to use (format: provider:model_name) |
| `--cloud-model` | Cloud model to use (format: provider:model_name) |
| `--cloud-api-key` | API key for the cloud model |

### Using the Server

Once started, the server will be available at:
- API endpoint: `http://localhost:<port>/v1/chat/completions`
- Health check: `http://localhost:<port>/health`

You can use any OpenAI-compatible client library:

```python
from openai import OpenAI

client = OpenAI(base_url="http://localhost:62549/v1", api_key="any-value")
response = client.chat.completions.create(
    model="auto",  # Use "auto" for Oblix's intelligent orchestration
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "What are the three laws of robotics?"}
    ]
)
print(response.choices[0].message.content)
```

## Chat Command

Start an interactive chat session with hybrid model orchestration:

```bash
# Start chat with specified models
oblix chat --local-model ollama:llama2 --cloud-model openai:gpt-3.5-turbo

# Use with Claude
oblix chat --local-model ollama:llama2 --cloud-model claude:claude-3-haiku --cloud-api-key YOUR_ANTHROPIC_KEY
```

### Chat Options

| Option | Description |
|--------|-------------|
| `--local-model` | Local model to use (format: provider:model_name) |
| `--cloud-model` | Cloud model to use (format: provider:model_name) |
| `--cloud-api-key` | API key for cloud model |

## Check for Updates

Check if a newer version of Oblix is available:

```bash
oblix check-updates
```

## Environment Variables

| Variable | Description |
|----------|-------------|
| `OPENAI_API_KEY` | Your OpenAI API key (used as default for OpenAI models) |
| `ANTHROPIC_API_KEY` | Your Anthropic API key (used as default for Claude models) |

## Troubleshooting

If you encounter issues with the CLI:

1. Ensure you're using the latest version of Oblix
2. Check that you've properly set up your API keys
3. Enable debug logging with the `--debug` flag
4. Verify that the Ollama server is running if using local models
5. Confirm your Mac meets the system requirements (particularly for larger models)
6. Check network connectivity if cloud models aren't responding

For additional help, join the Oblix Discord community at [https://discord.gg/v8qtEVuU](https://discord.gg/v8qtEVuU).