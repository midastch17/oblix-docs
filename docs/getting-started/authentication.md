# Authentication

To use the Oblix SDK, you'll need to authenticate using an Oblix API key. This guide explains how to obtain and use your API key.

## Getting an API Key

1. Create an account at [Oblix AI](https://oblix.ai)
2. Navigate to the API Keys section in your dashboard
3. Click "Create New API Key"
4. Copy your new API key to a secure location

!!! warning "API Key Security"
    Your Oblix API key grants access to the Oblix platform and should be kept secure. Never share your API key publicly or commit it to version control systems.

## Using Your API Key

There are two ways to provide your API key to the Oblix SDK:

### 1. Environment Variable (Recommended)

Set the `OBLIX_API_KEY` environment variable in a `.env` file:

```bash
# .env file
OBLIX_API_KEY=your_api_key_here
```

Then load it in your code:

```python
import os
from dotenv import load_dotenv
from oblix import OblixClient

# Load environment variables from .env file
load_dotenv()

# API key will be loaded from environment variable
oblix_api_key = os.getenv('OBLIX_API_KEY')
client = OblixClient(oblix_api_key=oblix_api_key)
```

### 2. Direct Initialization

Alternatively, you can pass your API key directly when initializing the client (not recommended for production code):

```python
from oblix import OblixClient

client = OblixClient(oblix_api_key="your_api_key_here")
```

## Provider API Keys

In addition to your Oblix API key, you'll need provider-specific API keys for any cloud models you want to use:

### OpenAI API Key

For using OpenAI models (like GPT-3.5 or GPT-4), you'll need to provide your OpenAI API key when hooking the model:

```python
import os
from dotenv import load_dotenv
from oblix import OblixClient, ModelType

# Load environment variables from .env file
load_dotenv()

oblix_api_key = os.getenv('OBLIX_API_KEY')
client = OblixClient(oblix_api_key=oblix_api_key)

# Hook an OpenAI model with your OpenAI API key from environment variable
openai_api_key = os.getenv('OPENAI_API_KEY')
await client.hook_model(
    model_type=ModelType.OPENAI,
    model_name="gpt-3.5-turbo",
    api_key=openai_api_key
)
```

### Anthropic API Key

For using Claude models, you'll need to provide your Anthropic API key:

```python
import os
from dotenv import load_dotenv
from oblix import OblixClient, ModelType

# Load environment variables from .env file
load_dotenv()

oblix_api_key = os.getenv('OBLIX_API_KEY')
client = OblixClient(oblix_api_key=oblix_api_key)

# Hook a Claude model with your Anthropic API key from environment variable
anthropic_api_key = os.getenv('ANTHROPIC_API_KEY')
await client.hook_model(
    model_type=ModelType.CLAUDE,
    model_name="claude-3-opus-20240229",
    api_key=anthropic_api_key
)
```

### Ollama (Local Models)

For Ollama models, no API key is required, but you'll need to specify the endpoint if you're not using the default:

```python
from oblix import OblixClient, ModelType

client = OblixClient(oblix_api_key="your_oblix_api_key")

# Hook an Ollama model
await client.hook_model(
    model_type=ModelType.OLLAMA,
    model_name="llama2",
    endpoint="http://localhost:11434"  # Optional, this is the default
)
```

## Troubleshooting

If you encounter authentication issues:

1. Verify that your API key is correct
2. Ensure you have an active Oblix subscription
3. Check that your API key has not been revoked
4. For provider-specific issues, verify your provider API keys

For persistent issues, contact Oblix support at [team@oblix.ai](mailto:team@oblix.ai).