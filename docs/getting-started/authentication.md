# Provider API Keys

To use Oblix with cloud-based LLM providers, you'll need to provide API keys for those services.


### OpenAI API Key

For using OpenAI models (like GPT-3.5 or GPT-4), you'll need to provide your OpenAI API key when hooking the model:

```python
import os
from dotenv import load_dotenv
from oblix import OblixClient, ModelType

# Load environment variables from .env file
load_dotenv()

client = OblixClient()

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

client = OblixClient()

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

client = OblixClient()

# Hook an Ollama model
await client.hook_model(
    model_type=ModelType.OLLAMA,
    model_name="llama2",
    endpoint="http://localhost:11434"  # Optional, this is the default
)
```

## Troubleshooting

If you encounter authentication issues:

1. Make sure your provider API keys are correct
2. Check that you have an active subscription with the LLM provider
3. Verify that the API keys have appropriate permissions

For persistent issues with cloud model providers, refer to their respective documentation:
- [OpenAI API Documentation](https://platform.openai.com/docs/api-reference)
- [Anthropic API Documentation](https://docs.anthropic.com/claude/reference/getting-started-with-the-api)
