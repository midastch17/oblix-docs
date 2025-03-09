# ModelType

The `ModelType` enum defines the supported AI model providers in the Oblix system. It's used when hooking models to specify which provider the model belongs to.

## Import

```python
from oblix import ModelType
```

## Enum Values

| Enum Value | Description |
|------------|-------------|
| `ModelType.OLLAMA` | Local models served via Ollama |
| `ModelType.OPENAI` | Cloud models provided by OpenAI (GPT series) |
| `ModelType.CLAUDE` | Cloud models provided by Anthropic (Claude series) |
| `ModelType.CUSTOM` | Custom model implementations |

## Usage

The `ModelType` enum is used primarily with the `hook_model()` method to specify which provider a model belongs to:

```python
from oblix import OblixClient, ModelType

client = OblixClient(oblix_api_key="your_api_key")

# Hook an Ollama model
await client.hook_model(
    model_type=ModelType.OLLAMA,
    model_name="llama2"
)

# Hook an OpenAI model
await client.hook_model(
    model_type=ModelType.OPENAI,
    model_name="gpt-3.5-turbo",
    api_key="your_openai_api_key"
)

# Hook a Claude model
await client.hook_model(
    model_type=ModelType.CLAUDE,
    model_name="claude-3-opus-20240229",
    api_key="your_anthropic_api_key"
)
```

## Required Parameters by Type

Each model type requires specific parameters:

### OLLAMA

- `model_name`: Name of the Ollama model (e.g., "llama2")
- `endpoint`: (Optional) API endpoint (default: "http://localhost:11434")

### OPENAI

- `model_name`: Name of the OpenAI model (e.g., "gpt-3.5-turbo")
- `api_key`: Your OpenAI API key

### CLAUDE

- `model_name`: Name of the Claude model (e.g., "claude-3-opus-20240229")
- `api_key`: Your Anthropic API key

### CUSTOM

- `model_name`: Name of your custom model
- Additional parameters as required by your implementation

## Supported Models

### OLLAMA

Ollama supports various open-source models, including:

- `llama2`
- `mixtral`
- `mistral`
- `phi`
- `gemma`
- And many others available through Ollama

### OPENAI

Supported OpenAI models include:

- `gpt-4-turbo`
- `gpt-4`
- `gpt-3.5-turbo`
- And other GPT models available through the OpenAI API

### CLAUDE

Supported Claude models include:

- `claude-3-opus-20240229`
- `claude-3-sonnet-20240229`
- `claude-3-haiku-20240307`
- And other Claude models available through the Anthropic API
