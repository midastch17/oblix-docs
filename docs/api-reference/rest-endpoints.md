# REST Endpoints API Reference

This page documents the HTTP REST API endpoints provided by the Oblix Server, particularly focusing on the OpenAI-compatible interface.

## Overview

Oblix Server provides several REST API endpoints for integration with external applications, including a fully compatible OpenAI API interface that allows you to use Oblix with any client library or tool that supports the OpenAI API.

## OpenAI-Compatible Endpoints

Oblix implements a set of endpoints that are compatible with the OpenAI API, making it easy to switch from direct OpenAI usage to Oblix's intelligent orchestration with minimal code changes.

### Base URL

The default Oblix server base URL for OpenAI-compatible endpoints is:

```
http://localhost:62549/v1
```

### Authentication

Use the `oblix-api-key` as the API key:

```python
from openai import OpenAI

client = OpenAI(
    base_url="http://localhost:62549/v1",
    api_key="oblix-api-key"  # The literal string "oblix-api-key"
)
```

### Chat Completions API

#### Endpoint: `/v1/chat/completions`

This endpoint is equivalent to OpenAI's Chat Completions API and is used to generate responses based on conversation history.

**Method:** POST

**Request Body:**

```json
{
  "model": "auto",
  "messages": [
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": "What are the three laws of robotics?"}
  ],
  "temperature": 0.7,
  "top_p": 1.0,
  "n": 1,
  "stream": false,
  "max_tokens": null,
  "presence_penalty": 0.0,
  "frequency_penalty": 0.0,
  "logit_bias": null,
  "user": null
}
```

**Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `model` | string | - | **Always use "auto"** for Oblix's intelligent orchestration to select the best model based on connectivity, system resources, and the specific request |
| `messages` | array | - | An array of message objects representing the conversation history |
| `temperature` | float | 0.7 | Controls randomness: Lowering results in less random completions. Range: 0.0 to 2.0 |
| `top_p` | float | 1.0 | Controls diversity via nucleus sampling: 0.5 means half of all likelihood-weighted options are considered |
| `n` | integer | 1 | Number of chat completion choices to generate for each input message |
| `stream` | boolean | false | If true, partial message deltas will be sent |
| `max_tokens` | integer | null | The maximum number of tokens to generate in the chat completion |
| `presence_penalty` | float | 0.0 | Number between -2.0 and 2.0. Positive values penalize new tokens based on whether they appear in the text so far |
| `frequency_penalty` | float | 0.0 | Number between -2.0 and 2.0. Positive values penalize new tokens based on their existing frequency in the text so far |
| `logit_bias` | object | null | Modify the likelihood of specified tokens appearing in the completion |
| `user` | string | null | A unique identifier representing your end-user, which can help Oblix monitor and detect abuse |

**Example Response:**

```json
{
  "id": "chatcmpl-abc123",
  "object": "chat.completion",
  "created": 1677858242,
  "model": "auto (selected: ollama:llama2)",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "The Three Laws of Robotics as formulated by Isaac Asimov are:\n\n1. A robot may not injure a human being or, through inaction, allow a human being to come to harm.\n\n2. A robot must obey the orders given it by human beings except where such orders would conflict with the First Law.\n\n3. A robot must protect its own existence as long as such protection does not conflict with the First or Second Law."
      },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 30,
    "completion_tokens": 97,
    "total_tokens": 127
  }
}
```

### Streaming Mode

Oblix supports streaming responses by setting the `stream` parameter to `true`. In streaming mode, the API will return a stream of events as the response is being generated.

**Example Python Code with Streaming:**

```python
from openai import OpenAI

client = OpenAI(
    base_url="http://localhost:62549/v1",
    api_key="oblix-api-key"
)

stream = client.chat.completions.create(
    model="auto",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "What is quantum computing?"}
    ],
    stream=True,
    temperature=0.7
)

for chunk in stream:
    if chunk.choices[0].delta.content is not None:
        print(chunk.choices[0].delta.content, end="")
```

## Using Oblix Server in Production

When integrating Oblix into production environments, consider the following best practices:

### 1. Connection Management

For high-traffic applications, maintain persistent connections to the Oblix server to avoid connection establishment overhead.

### 2. Error Handling

Implement robust error handling to manage potential issues:

```python
try:
    response = client.chat.completions.create(
        model="auto",
        messages=[{"role": "user", "content": "Hello, world!"}],
        temperature=0.7
    )
except Exception as e:
    # Handle the error appropriately
    print(f"Error communicating with Oblix server: {e}")
```

### 3. Model Selection Strategy

While Oblix's `auto` model selection is recommended for most use cases, understand the orchestration strategy to ensure it meets your specific requirements.

### 4. Load Balancing

For high-demand scenarios, consider deploying multiple Oblix servers behind a load balancer.

## Examples

### Basic Usage

```python
from openai import OpenAI

client = OpenAI(
    base_url="http://localhost:62549/v1",
    api_key="oblix-api-key"
)

response = client.chat.completions.create(
    model="auto",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "What is the capital of France?"}
    ],
    temperature=0.7
)

print(response.choices[0].message.content)
```

### Setting Different Temperatures

```python
# For more deterministic responses
response = client.chat.completions.create(
    model="auto",
    messages=[{"role": "user", "content": "Write a poem about autumn."}],
    temperature=0.2  # Lower temperature for more focused, deterministic output
)

# For more creative responses
response = client.chat.completions.create(
    model="auto",
    messages=[{"role": "user", "content": "Write a poem about autumn."}],
    temperature=1.2  # Higher temperature for more diverse, creative output
)
```

### Limiting Response Length

```python
response = client.chat.completions.create(
    model="auto",
    messages=[{"role": "user", "content": "Explain the history of computing."}],
    max_tokens=100  # Limit response to approximately 100 tokens
)
```

### Adjusting Creativity with Top-p

```python
response = client.chat.completions.create(
    model="auto",
    messages=[{"role": "user", "content": "Generate a story idea."}],
    top_p=0.5  # Only consider the top 50% of probability mass for each token
)
```

## Next Steps

- Learn about [OblixClient API](oblix-client.md) for direct SDK integration
- Explore [Integration Guides](../integration-guides/index.md) for frontend and server integration strategies
- Check out [Examples](../examples/index.md) for more usage scenarios