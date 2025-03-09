# CLI Reference

Oblix provides a powerful command-line interface (CLI) that enables you to interact with the Oblix AI Orchestration SDK directly from your terminal. This tool simplifies model management, agent integration, and testing.

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

## Available Commands

| Command | Description |
|---------|-------------|
| `models` | Manage and interact with AI models |
| `agents` | Manage and interact with Oblix agents |
| `sessions` | Manage chat sessions |
| `execute` | Execute a single prompt |
| `chat` | Start an interactive chat session |

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

Commands for managing AI models:

### List Models

List all configured models:

```bash
oblix models list
```

### Hook Model

Add a new model to Oblix:

```bash
# Hook an Ollama model
oblix models hook --type ollama --name llama2

# Hook an OpenAI model
oblix models hook --type openai --name gpt-3.5-turbo --api-key-model YOUR_OPENAI_KEY

# Hook a Claude model
oblix models hook --type claude --name claude-3-haiku-20240307 --api-key-model YOUR_ANTHROPIC_KEY
```

### Model Info

Get information about a specific model:

```bash
oblix models info --type ollama --name llama2
```

## Agents Commands

Commands for interacting with Oblix agents:

### Connectivity

Check current connectivity metrics:

```bash
oblix agents connectivity
```

### Resource

Perform a system resource check:

```bash
oblix agents resource
```

## Sessions Commands

Commands for managing chat sessions:

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

## Execute and Chat Commands

Execute a single prompt or start an interactive chat:

### Execute a Prompt

```bash
oblix execute "Explain quantum computing in simple terms"
```

With specific model:
```bash
oblix execute "Explain quantum computing" --model "openai:gpt-3.5-turbo"
```

### Start Interactive Chat

Start an interactive chat session with automatic model orchestration:

```bash
oblix chat --local-model llama2 --cloud-model gpt-3.5-turbo
```

With Claude:
```bash
oblix chat --local-model llama2 --cloud-model claude-3-haiku --cloud-api-key YOUR_ANTHROPIC_KEY
```

## Examples

Here are some common use cases for the Oblix CLI:

### Setting Up Hybrid Model Orchestration

```bash
# Set up local and cloud models for orchestration
oblix models hook --type ollama --name llama2
oblix models hook --type openai --name gpt-3.5-turbo --api-key-model YOUR_OPENAI_KEY

# Start interactive chat with model orchestration
oblix chat --local-model llama2 --cloud-model gpt-3.5-turbo
```

### Checking System Status

```bash
# Check system resource metrics
oblix agents resource

# Check connectivity metrics
oblix agents connectivity
```

### Managing Sessions

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

## Troubleshooting

If you encounter issues with the CLI:

1. Ensure you're using the latest version of Oblix
2. Check that you've properly set up your API keys
3. Enable debug logging with the `--debug` flag
4. Verify that the Ollama server is running if using local models
5. Confirm your Mac meets the system requirements (particularly for larger models)

For additional help, join the Oblix Discord community at [https://discord.gg/v8qtEVuU](https://discord.gg/v8qtEVuU).