# CLI Reference

Oblix provides a powerful command-line interface (CLI) that enables you to interact with the Oblix AI Orchestration SDK directly from your terminal. This tool simplifies hybrid model orchestration between cloud and edge environments.

## Installation

The Oblix CLI is included with the main Oblix SDK package when you download and install it from [oblix.ai](https://oblix.ai).

> **Note:** Currently, Oblix only supports macOS. Windows and Linux versions are planned for future releases.

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
| `models` | Manage and interact with AI models (both local and cloud) |
| `agents` | Manage and interact with Oblix orchestration agents |
| `sessions` | Manage chat sessions |
| `execute` | Execute a single prompt with orchestration |
| `chat` | Start an interactive chat session with hybrid model orchestration |

## Authentication

Most commands require an Oblix API key, which can be provided in several ways:

1. Using the `--api-key` option with each command
2. Setting the `OBLIX_API_KEY` environment variable
3. When not provided, the CLI will prompt you to enter it

Example:
```bash
export OBLIX_API_KEY="your_oblix_api_key"
```

## Common Options

These options are available for most commands:

| Option | Description |
|--------|-------------|
| `--help`, `-h` | Display help for a command |
| `--version` | Show the Oblix CLI version |
| `--debug` | Enable debug logging |

## Models Commands

Commands for managing AI models across cloud and edge:

### List Models

List all configured models:

```bash
oblix models list
```

### Hook Model

Add a new model to the orchestration system:

```bash
# Hook a local Ollama model
oblix models hook --type ollama --name llama2

# Hook a cloud OpenAI model
oblix models hook --type openai --name gpt-3.5-turbo --api-key-model YOUR_OPENAI_KEY

# Hook a cloud Claude model
oblix models hook --type claude --name claude-3-haiku-20240307 --api-key-model YOUR_ANTHROPIC_KEY
```

### Model Info

Get information about a specific model:

```bash
oblix models info --type ollama --name llama2
```

## Agents Commands

Commands for interacting with Oblix orchestration agents:

### Connectivity

Check current connectivity metrics (critical for orchestration decisions):

```bash
oblix agents connectivity
```

### Resource

Perform a system resource check (impacts model routing decisions):

```bash
oblix agents resource
```

## Sessions Commands

Commands for managing orchestrated chat sessions:

### List Sessions

List recent chat sessions:

```bash
oblix sessions list
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

## Orchestrated Execution and Chat

Execute prompts with automatic orchestration between cloud and edge:

### Execute a Prompt with Orchestration

Let Oblix decide which model to use based on system state:

```bash
oblix execute "Explain quantum computing in simple terms"
```

With a specific model (bypassing orchestration):
```bash
oblix execute "Explain quantum computing" --model "openai:gpt-3.5-turbo"
```

### Start Interactive Chat with Hybrid Orchestration

Start an interactive chat session with automatic orchestration between cloud and edge models:

```bash
oblix chat --local-model llama2 --cloud-model gpt-3.5-turbo
```

With Claude as the cloud model:
```bash
oblix chat --local-model llama2 --cloud-model claude-3-haiku --cloud-api-key YOUR_ANTHROPIC_KEY
```

## Orchestration Examples

Here are some common orchestration use cases for the Oblix CLI:

### Setting Up Hybrid Model Orchestration

```bash
# Set up local and cloud models for orchestration
oblix models hook --type ollama --name llama2
oblix models hook --type openai --name gpt-3.5-turbo --api-key-model YOUR_OPENAI_KEY

# Start interactive chat with model orchestration
oblix chat --local-model llama2 --cloud-model gpt-3.5-turbo
```

### Checking Orchestration System Status

```bash
# Check system resource metrics (affects local model selection)
oblix agents resource

# Check connectivity metrics (affects cloud model selection)
oblix agents connectivity
```

### Managing Orchestrated Sessions

```bash
# List recent sessions
oblix sessions list

# Create a new session
oblix sessions create --title "Research Project"

# Execute with a specific session
oblix execute "Research quantum computing papers" --session-id YOUR_SESSION_ID
```

## Environment Variables

| Variable | Description |
|----------|-------------|
| `OBLIX_API_KEY` | Your Oblix API key |
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