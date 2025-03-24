# Troubleshooting Guide

This guide covers common issues that may occur when working with Oblix, along with their solutions.

## JSON Serialization Errors with Infinity Values

### Issue

When working with Oblix in disconnected mode, you might encounter errors like:

```
ValueError: Out of range float values are not JSON compliant
```

### Solution

This occurs because Python's `float('inf')` values (used in connectivity metrics) cannot be directly serialized to JSON. Implement a sanitization function for metrics before returning them in your API:

```python
def sanitize_metrics(m):
    """Replace non-JSON-compliant float values with string representations."""
    if isinstance(m, dict):
        return {k: sanitize_metrics(v) for k, v in m.items()}
    elif isinstance(m, list):
        return [sanitize_metrics(i) for i in m]
    elif isinstance(m, float):
        if m == float('inf'):
            return "Infinity"
        elif m == float('-inf'):
            return "-Infinity"
        elif m != m:  # check for NaN
            return None
        return m
    return m

# Then use this function before returning metrics
metrics = sanitize_metrics(metrics)
```

## Ollama Connection Issues

### Issue

Ollama models fail to connect with errors like:

```
Connection refused: connect to Ollama server
```

### Solution

1. Make sure Ollama is installed and running:
   ```bash
   # Check if Ollama is running
   curl http://localhost:11434/api/version
   ```

2. If Ollama is not running, start it:
   ```bash
   # Start Ollama service
   ollama serve
   ```

3. Check if your desired model is pulled:
   ```bash
   # List installed models
   ollama list
   
   # Pull a model if needed
   ollama pull llama2
   ```

4. Verify the endpoint in your configuration (default is `http://localhost:11434`)

## Model Selection Not Working as Expected

### Issue

Oblix doesn't select the expected model (e.g., using Ollama when OpenAI should be preferred).

### Solution

1. Check your current connectivity state:
   ```python
   metrics = await client.get_connectivity_metrics()
   print(metrics)
   ```

2. Verify resource monitor measurements:
   ```python
   resources = await client.get_resource_metrics()
   print(resources)
   ```

3. Review the routing decision from a response:
   ```python
   result = await client.execute("Test message")
   print("Routing decision:", result.get("routing_decision"))
   ```

4. Adjust connectivity thresholds if needed:
   ```python
   connectivity_agent = ConnectivityAgent(
       latency_threshold=150.0,       # Lower for more cloud use
       packet_loss_threshold=5.0,     # Lower for more cloud use
       bandwidth_threshold=10.0       # Higher for more cloud use
   )
   ```

## Session Management Issues

### Issue

Session data is lost between API calls or application restarts.

### Solution

1. Always explicitly specify session_id when making API calls:
   ```python
   # Create a session once
   session_id = await client.create_session(title="My Chat")
   
   # Use the same session_id for subsequent calls
   result = await client.execute("Hello", session_id=session_id)
   ```

2. For FastAPI implementations, store the session_id on the client side and send it with each request:
   ```javascript
   // Store session ID in localStorage
   localStorage.setItem('oblixSessionId', sessionId);
   
   // Retrieve for later use
   const storedSessionId = localStorage.getItem('oblixSessionId');
   ```

3. Check if sessions exist before using them:
   ```python
   sessions = client.list_sessions()
   if session_id not in [s['id'] for s in sessions]:
       # Create a new session if the old one doesn't exist
       session_id = await client.create_session(title="New Chat")
   ```

## Memory Usage Issues with Large Models

### Issue

Local Ollama models cause high memory usage or crashes.

### Solution

1. Use models with smaller parameter counts:
   ```python
   # Instead of large models like llama2:70b, use smaller variants
   await client.hook_model(
       model_type=ModelType.OLLAMA,
       model_name="llama2:7b",  # 7B parameter model instead of larger ones
       endpoint="http://localhost:11434"
   )
   ```

2. Enable Oblix's resource monitoring to automatically switch to cloud when resources are constrained:
   ```python
   resource_monitor = ResourceMonitor(
       cpu_threshold=80.0,        # CPU % threshold
       memory_threshold=85.0,     # Memory % threshold
       gpu_threshold=90.0         # GPU memory % threshold (if applicable)
   )
   client.hook_agent(resource_monitor)
   ```

3. Lower Ollama's GPU memory usage (if using GPU):
   ```bash
   # Start Ollama with limited GPU memory
   CUDA_VISIBLE_DEVICES=0 ollama serve
   ```

## Handling Offline Mode

### Issue

Application crashes or provides poor user experience when internet is disconnected.

### Solution

1. Make sure you have both local and cloud models hooked:
   ```python
   # Hook local model for offline use
   await client.hook_model(
       model_type=ModelType.OLLAMA,
       model_name="llama2",
       endpoint="http://localhost:11434"
   )
   
   # Hook cloud model for online use
   await client.hook_model(
       model_type=ModelType.OPENAI,
       model_name="gpt-3.5-turbo",
       api_key=openai_api_key
   )
   ```

2. Add the connectivity agent to detect connection changes:
   ```python
   connectivity_agent = ConnectivityAgent(name="connectivity_monitor")
   client.hook_agent(connectivity_agent)
   ```

3. In your UI, handle API errors gracefully:
   ```javascript
   try {
     const response = await api.sendMessage(message, sessionId);
     // Process successful response
   } catch (error) {
     // Display user-friendly error
     if (error.message.includes('disconnected') || error.message.includes('network')) {
       showUserMessage("Operating in offline mode with local models only.");
     } else {
       showUserMessage(`Error: ${error.message}`);
     }
   }
   ```

## API Key Validation Errors

### Issue

API key validation errors when initializing the Oblix client.

### Solution

1. Check the API key format (should start with `obx_sk_`):
   ```python
   import re
   if not re.match(r'^obx_sk_[A-Za-z0-9_\-\.]{20,60}$', api_key):
       print("Invalid API key format")
   ```

2. Ensure the API key is properly set in environment variables:
   ```python
   # Set the API key environment variable
   import os
   os.environ['OBLIX_API_KEY'] = 'obx_sk_your_key_here'
   
   # Initialize client without explicitly passing key
   client = OblixClient()  # Will use env var
   ```

3. Test API key validation directly:
   ```python
   from oblix.auth import OblixAuth
   
   async def validate_key(api_key):
       auth = OblixAuth(api_key=api_key)
       try:
           await auth.validate_key(force_revalidation=True)
           print("API key is valid")
           return True
       except Exception as e:
           print(f"API key validation failed: {e}")
           return False
   ```

## Async/Event Loop Errors

### Issue

Errors related to async event loops when using Oblix.

### Solution

1. Use the correct async pattern in your application:
   ```python
   async def main():
       client = OblixClient(oblix_api_key=api_key)
       # Use client async methods here
       await client.shutdown()
   
   if __name__ == "__main__":
       import asyncio
       asyncio.run(main())
   ```

2. For FastAPI, ensure proper async handling:
   ```python
   @app.post("/api/oblix/execute")
   async def execute_prompt(request: dict):
       global oblix_client
       if not oblix_client:
           return {"error": "Client not initialized"}
       
       try:
           result = await oblix_client.execute(request["prompt"])
           return result
       except Exception as e:
           return {"error": str(e)}
   ```

3. For loops using client in existing event loop:
   ```python
   try:
       loop = asyncio.get_running_loop()
   except RuntimeError:
       # No running event loop, create one
       loop = asyncio.new_event_loop()
       asyncio.set_event_loop(loop)
       result = loop.run_until_complete(client.execute("prompt"))
   else:
       # Use existing event loop
       result = loop.run_until_complete(client.execute("prompt"))
   ```

## Client Shutdown Issues

### Issue

Resources not properly cleaned up when the application exits.

### Solution

1. Always call shutdown when done:
   ```python
   try:
       # Your code here
   finally:
       # Ensure client shutdown happens
       if client:
           loop = asyncio.get_event_loop()
           loop.run_until_complete(client.shutdown())
   ```

2. For FastAPI, implement proper shutdown handling:
   ```python
   @app.on_event("shutdown")
   async def shutdown_event():
       global oblix_client
       if oblix_client:
           try:
               await oblix_client.shutdown()
               logger.info("Oblix client shutdown successful")
           except Exception as e:
               logger.error(f"Error shutting down Oblix client: {e}")
   ```

3. For Electron applications, ensure process termination:
   ```javascript
   app.on('quit', () => {
     if (backendProcess) {
       // Terminate the backend process
       if (process.platform === 'win32') {
         spawn('taskkill', ['/pid', backendProcess.pid, '/f', '/t']);
       } else {
         backendProcess.kill();
       }
     }
   });
   ```

## Still Having Issues?

If you're still experiencing problems:

1. Check the Oblix logs for more detailed error information
2. Upgrade to the latest version of Oblix: `pip install oblix --upgrade`
3. For API key issues, contact Oblix support to verify your account status
4. For model-specific issues, check the provider's status page (OpenAI, Anthropic, etc.)
5. Report issues on the [Oblix GitHub repository](https://github.com/oblix-ai/oblix)