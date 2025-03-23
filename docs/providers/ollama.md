# Ollama

Ollama is an open-source tool that allows you to run open-source large language models locally on your own hardware. This page covers how to use Ollama models with Oblix for intelligent model orchestration.

## Installation and Setup

### 1. Install Oblix

Oblix is available exclusively for macOS at this time:

1. Visit [oblix.ai](https://oblix.ai) to download the latest version
2. Follow the installation instructions on the website
3. Once installed, you can import the SDK in your Python projects

> **Note:** Currently, only macOS is supported. Windows and Linux versions are planned for future releases.

### 2. Install Ollama

Before using Ollama with Oblix, you need to install Ollama on your Mac:

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

### 3. Start Ollama Server

Once installed, make sure the Ollama server is running:

```bash
ollama serve
```

This starts the Ollama server on the default port `11434`.

### 4. Pull Models

Pull the models you want to use:

```bash
# Examples of models you can pull
ollama pull llama2
ollama pull mistral
ollama pull gemma
```

## Hooking Ollama Models in Oblix

```python
from oblix import OblixClient, ModelType

# Initialize client
client = OblixClient(oblix_api_key="your_oblix_api_key")

# Hook Ollama model with default endpoint (http://localhost:11434)
await client.hook_model(
    model_type=ModelType.OLLAMA,
    model_name="llama2"
)

# Or specify a custom endpoint
await client.hook_model(
    model_type=ModelType.OLLAMA,
    model_name="llama2",
    endpoint="http://custom-server:11434"
)
```

## Supported Ollama Models

Ollama supports a wide range of open-source models, including:

| Model Family | Examples | Description |
|--------------|----------|-------------|
| Llama | llama2, llama3 | Meta's Llama models |
| Mistral | mistral, mixtral | Mistral AI's models |
| Phi | phi-2 | Microsoft's Phi models |
| Gemma | gemma:2b, gemma:7b | Google's Gemma models |
| Vicuna | vicuna | Fine-tuned Llama models |
| Orca | orca-mini | Microsoft's Orca models |
| Falcon | falcon | TII's Falcon models |

For the most up-to-date list of available models, see the [Ollama model library](https://ollama.com/library).

## Using Ollama with Oblix

Ollama works best as part of a hybrid setup with Oblix, where you combine local and cloud models for optimal results. While it's technically possible to use just Ollama models, Oblix is designed to excel in orchestration scenarios where both local and cloud models are available.

### Hybrid Local-Cloud Setup with Ollama

```python
from oblix import OblixClient, ModelType
from oblix.agents.resource_monitor import ResourceMonitor
from oblix.agents.connectivity import ConnectivityAgent

# Initialize client
client = OblixClient(oblix_api_key="your_oblix_api_key")

# Hook local Ollama model
await client.hook_model(
    model_type=ModelType.OLLAMA,
    model_name="llama2"
)

# Hook cloud model
await client.hook_model(
    model_type=ModelType.OPENAI,
    model_name="gpt-3.5-turbo",
    api_key="your_openai_api_key"
)

# Add monitoring agents for intelligent orchestration
client.hook_agent(ResourceMonitor())
client.hook_agent(ConnectivityAgent())

# Execute prompt - Oblix will automatically choose between 
# local and cloud models based on connectivity and system resources
response = await client.execute("Explain quantum computing in simple terms")
print(response["response"])
```

## Advanced Configuration

You can customize Ollama model settings when executing prompts:

```python
response = await client.execute(
    "Write a story about a robot who gains consciousness",
    temperature=0.8,
    max_tokens=2000,
    use_gpu=True  # Enable GPU acceleration if available
)
```

### Supported Parameters

Ollama models in Oblix support the following parameters:

| Parameter | Description | Default |
|-----------|-------------|---------|
| `temperature` | Controls randomness (0-2) | 0.7 |
| `max_tokens` | Maximum response length | model-dependent |
| `use_gpu` | Enable GPU acceleration | False |
| `stop` | List of stop sequences | None |

## Performance Considerations

When using Ollama models with Oblix, consider:

### System Requirements for macOS

Models have different resource requirements on Mac:

| Model Size | RAM Required | Disk Space | Apple Silicon Benefits |
|------------|--------------|------------|------------------------|
| 7B | 8GB+ | ~4GB | Good performance on M1/M2 |
| 13B | 16GB+ | ~8GB | Better with M1 Pro/Max/Ultra or M2 |
| 70B | 32GB+ | ~40GB | Best with M1/M2 Max/Ultra |

### Apple Silicon Optimization

Ollama is optimized for Apple Silicon, and Oblix leverages this effectively:

- **Metal API**: Hardware acceleration on Apple Silicon chips
- **Memory management**: Optimized for macOS memory architecture
- **Resource monitoring**: Oblix's ResourceMonitor is specially tuned for macOS

### Offline Capability

One of the key advantages of Ollama models is offline capability:

- Models run completely locally without internet connectivity
- Perfect for privacy-sensitive applications
- Ideal for environments with unreliable connections

### Oblix Orchestration Benefits

The combination of Ollama and Oblix provides unique advantages:

- Intelligent switching between local Ollama models and cloud models
- Automatic detection of connectivity status and resource availability
- Optimization for macOS-specific hardware features
- Seamless fallback to local models when offline

## Troubleshooting

Common issues and their solutions:

### Model Not Found

If you encounter "model not found" errors:

```bash
# Pull the model manually first
ollama pull llama2
```

### Port Issues

If Ollama is running on a non-standard port:

```python
await client.hook_model(
    model_type=ModelType.OLLAMA,
    model_name="llama2",
    endpoint="http://localhost:YOUR_PORT"
)
```

### Resource Constraints on Mac

If the model is running slowly or crashing on your Mac:

- Try a smaller model (e.g., switch from llama2:13b to llama2:7b)
- Ensure adequate RAM is available
- Close other memory-intensive applications
- For M1/M2 Macs, check Activity Monitor for memory pressure

### MacOS-Specific Optimization

To get the best performance with Ollama on macOS:

- Ensure the latest version of macOS is installed
- Keep Ollama updated for latest Metal API optimizations
- For Apple Silicon Macs, use models optimized for Metal (e.g., llama2:7b-q4_0)
- Oblix's ResourceMonitor will automatically detect Metal compatibility

## API Reference

See the [OblixClient API reference](../api-reference/oblix-client.md) for detailed information on all available methods.