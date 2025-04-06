# FastAPI Server Integration

This guide shows how to create a robust FastAPI server that uses Oblix for intelligent model orchestration, providing a simple HTTP API for your frontend applications.

## Why FastAPI?

FastAPI is an excellent choice for building an Oblix backend service because:

- **Modern async support** - Works well with Oblix's async design
- **Strong typing** - Helps catch errors early with Pydantic schema validation
- **Automatic documentation** - Swagger UI for easy API exploration
- **High performance** - Built on Starlette and Uvicorn for speed
- **Error handling** - Robust exception handling and clean error responses

## Prerequisites

- Python 3.9+
- Oblix SDK installed (`pip install oblix`)
- FastAPI and dependencies (`pip install fastapi uvicorn`)

## Basic FastAPI Server

Here's a minimal implementation of an Oblix FastAPI server:

```python
#!/usr/bin/env python3
import asyncio
import json
import logging
import os
import socket
import time
from typing import Dict, List, Optional, Union

import uvicorn
from fastapi import FastAPI, HTTPException
from fastapi.middleware.cors import CORSMiddleware
from fastapi.responses import JSONResponse
from pydantic import BaseModel, Field

# Import Oblix SDK
from oblix import OblixClient, ModelType
from oblix.agents import ResourceMonitor, ConnectivityAgent

# Configure logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)
logger = logging.getLogger("oblix-fastapi")

# Initialize FastAPI app
app = FastAPI(title="Oblix FastAPI Server")

# Add CORS middleware
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],  # For production, restrict to your frontend domains
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Global client
oblix_client = None
active_models = {}

# Pydantic models for request validation
class ModelConfig(BaseModel):
    model_type: str
    model_name: str
    endpoint: Optional[str] = None
    api_key: Optional[str] = None

class InitializeRequest(BaseModel):
    models: List[ModelConfig]

class ChatRequest(BaseModel):
    message: str
    session_id: Optional[str] = None
    temperature: float = 0.7
    max_tokens: Optional[int] = None
    api_key: Optional[str] = None

@app.get("/api/health")
async def health_check():
    """Health check endpoint."""
    return {
        "status": "healthy",
        "timestamp": time.strftime("%Y-%m-%d %H:%M:%S"),
        "version": "0.1.0",
        "service": "oblix-fastapi"
    }

@app.post("/api/oblix/initialize")
async def initialize_oblix(request: InitializeRequest):
    """Initialize the Oblix client with API keys and models."""
    global oblix_client, active_models
    
    try:
        # Shutdown existing client if it exists
        if oblix_client:
            try:
                await oblix_client.shutdown()
            except Exception as e:
                logger.error(f"Error shutting down existing client: {e}")
        
        # Create a new client
        oblix_client = OblixClient()
        
        # Reset active models
        active_models = {
            "ollama": None,
            "openai": None,
            "anthropic": None
        }
        
        # Hook models
        for model_config in request.models:
            try:
                if model_config.model_type.upper() == "OLLAMA":
                    # Hook Ollama model
                    await oblix_client.hook_model(
                        model_type=ModelType.OLLAMA,
                        model_name=model_config.model_name,
                        endpoint=model_config.endpoint or "http://localhost:11434"
                    )
                    active_models["ollama"] = model_config.model_name
                    
                elif model_config.model_type.upper() == "OPENAI":
                    # Hook OpenAI model
                    await oblix_client.hook_model(
                        model_type=ModelType.OPENAI,
                        model_name=model_config.model_name,
                        api_key=model_config.api_key
                    )
                    active_models["openai"] = model_config.model_name
                    
                elif model_config.model_type.upper() in ["CLAUDE", "ANTHROPIC"]:
                    # Hook Claude model
                    await oblix_client.hook_model(
                        model_type=ModelType.CLAUDE,
                        model_name=model_config.model_name,
                        api_key=model_config.api_key
                    )
                    active_models["anthropic"] = model_config.model_name
            except Exception as e:
                logger.error(f"Error hooking {model_config.model_type} model: {e}")
        
        # Add monitoring agents
        resource_monitor = ResourceMonitor(name="resource_monitor")
        connectivity_monitor = ConnectivityAgent(name="connectivity_monitor")
        oblix_client.hook_agent(resource_monitor)
        oblix_client.hook_agent(connectivity_monitor)
        
        # Create a default session
        await oblix_client.create_session(title="Default Chat")
        
        return {
            "status": "success",
            "message": "Oblix client initialized successfully",
            "active_models": active_models
        }
    
    except Exception as e:
        logger.error(f"Error initializing Oblix client: {e}")
        return JSONResponse(
            status_code=500,
            content={
                "status": "error",
                "message": f"Failed to initialize Oblix client: {str(e)}"
            }
        )

@app.post("/api/oblix/create-session")
async def create_session(request: dict):
    """Create a new Oblix session."""
    global oblix_client
    
    if not oblix_client:
        return JSONResponse(
            status_code=400,
            content={
                "status": "error",
                "message": "Oblix client not initialized"
            }
        )
    
    try:
        title = request.get("title", "New Chat")
        session_id = await oblix_client.create_session(title=title)
        return {
            "status": "success",
            "sessionId": session_id
        }
    except Exception as e:
        logger.error(f"Error creating session: {e}")
        return JSONResponse(
            status_code=500,
            content={
                "status": "error",
                "message": f"Failed to create session: {str(e)}"
            }
        )

@app.post("/api/oblix/stream-chat")
async def stream_chat(request: ChatRequest):
    """Handle chat messages with Oblix."""
    global oblix_client
    
    if not oblix_client:
        return JSONResponse(
            status_code=400,
            content={
                "status": "error",
                "message": "Oblix client not initialized"
            }
        )
    
    try:
        # Use existing session or create a new one
        session_id = request.session_id
        if not session_id:
            session_id = await oblix_client.create_session(title="New Chat")
        
        # Set current session
        oblix_client.use_session(session_id)
        
        # Execute the message
        start_time = time.time()
        result = await oblix_client.execute(
            request.message,
            temperature=request.temperature,
            max_tokens=request.max_tokens
        )
        end_time = time.time()
        
        # Get model information
        model_id = result.get("model_id", "unknown")
        metrics = result.get("metrics", {})
        metrics["latency"] = end_time - start_time
        
        # Sanitize metrics to prevent JSON serialization errors
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
        
        metrics = sanitize_metrics(metrics)
        
        # If connectivity_metrics exists, also sanitize it
        if "connectivity_metrics" in result:
            result["connectivity_metrics"] = sanitize_metrics(result["connectivity_metrics"])
        
        # Get response text
        response_text = result.get("response", "")
        if not isinstance(response_text, str):
            response_text = str(response_text)
        
        return {
            "status": "success",
            "response": response_text,
            "sessionId": session_id,
            "model_id": model_id,
            "metrics": metrics,
            "agent_checks": result.get("agent_checks", []),
            "routing_decision": result.get("routing_decision", {})
        }
    
    except Exception as e:
        logger.error(f"Error processing chat: {e}")
        return JSONResponse(
            status_code=500,
            content={
                "status": "error",
                "message": f"Failed to process chat: {str(e)}"
            }
        )

def start_server(port=8000):
    """Start the FastAPI server."""
    uvicorn.run(app, host="127.0.0.1", port=port, log_level="info")

if __name__ == "__main__":
    start_server()
```

## Important Implementation Details

### 1. Handling Special Float Values

When working with Oblix in disconnected mode, the connectivity metrics can include `float('inf')` values that are not JSON serializable. To handle this, implement a sanitization function:

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
```

### 2. Managing the Oblix Client Lifecycle

The Oblix client should be properly initialized and shutdown:

```python
# Shutdown existing client before creating a new one
if oblix_client:
    try:
        await oblix_client.shutdown()
    except Exception as e:
        logger.error(f"Error shutting down existing client: {e}")

# Proper shutdown on application exit
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

### 3. Session Management

The server should handle session management, including:
- Creating sessions when needed
- Verifying existing sessions
- Properly maintaining context

Here's an improved implementation for handling sessions:

```python
@app.post("/api/oblix/stream-chat")
async def stream_chat(request: ChatRequest):
    global oblix_client
    
    if not oblix_client:
        return JSONResponse(
            status_code=400,
            content={
                "status": "error",
                "message": "Oblix client not initialized"
            }
        )
    
    try:
        # Use existing session or create a new one
        session_id = request.session_id
        
        # Verify the session exists or create a new one
        try:
            # Check if session exists
            sessions = oblix_client.list_sessions()
            session_exists = any(s.get('id') == session_id for s in sessions)
            
            if not session_id or not session_exists:
                # Create a new session if needed
                session_id = await oblix_client.create_session(title="New Chat")
                logger.info(f"Created new session: {session_id}")
            else:
                # Session exists, let's load it properly
                logger.info(f"Reusing existing session: {session_id}")
                
                # Explicitly load the session to ensure we have all the context
                session_data = oblix_client.load_session(session_id)
                if session_data:
                    logger.info(f"Successfully loaded session {session_id}")
                else:
                    logger.warning(f"Failed to load session data for {session_id}")
            
            # Set current session
            oblix_client.use_session(session_id)
            logger.info(f"Set active session to {session_id}")
        except Exception as e:
            logger.warning(f"Session handling error: {e}")
            # Create a new session as fallback
            session_id = await oblix_client.create_session(title="New Chat")
            oblix_client.use_session(session_id)
            logger.info(f"Created fallback session: {session_id}")
        
        # Execute with the established session context
        result = await oblix_client.execute(
            request.message,
            temperature=request.temperature,
            max_tokens=request.max_tokens
        )
        
        # Return response with session ID
        return {
            "status": "success",
            "response": result.get("response", ""),
            "sessionId": session_id,
            # ...other response fields
        }
    except Exception as e:
        logger.error(f"Error processing chat: {e}")
        # Error handling...
```

This implementation ensures proper context preservation when switching between chat sessions.

## Additional Endpoints

For a complete solution, consider adding these endpoints:

### Ollama Status Check

```python
@app.post("/api/ollama/status")
async def get_ollama_status(request: dict):
    """Check if Ollama is running."""
    try:
        endpoint = request.get('endpoint', 'http://localhost:11434')
        import requests
        response = requests.get(f"{endpoint}/api/version", timeout=5)
        return {
            "status": "success" if response.ok else "error",
            "running": response.ok,
            "message": "Ollama is running" if response.ok else "Ollama is not running"
        }
    except Exception as e:
        return {
            "status": "error",
            "running": False,
            "message": f"Could not connect to Ollama: {str(e)}"
        }
```

### List Available Ollama Models

```python
@app.post("/api/ollama/models")
async def check_ollama_models(request: dict):
    """Check Ollama models."""
    try:
        endpoint = request.get('endpoint', 'http://localhost:11434')
        import requests
        response = requests.get(f"{endpoint}/api/tags", timeout=5)
        
        if not response.ok:
            return {
                "status": "error",
                "message": f"Failed to get models: HTTP {response.status_code}",
                "models": []
            }
            
        models_data = response.json()
        models = []
        
        # Process models based on response format
        if 'models' in models_data:
            for model in models_data['models']:
                model_name = model.get('name', 'unknown')
                models.append({
                    "id": f"ollama-{model_name}",
                    "name": model_name
                })
        elif isinstance(models_data, list):
            for model in models_data:
                if isinstance(model, dict) and 'name' in model:
                    model_name = model.get('name', 'unknown')
                    models.append({
                        "id": f"ollama-{model_name}",
                        "name": model_name
                    })
        
        return {
            "status": "success",
            "message": "Ollama connected successfully",
            "models": models
        }
    except Exception as e:
        return {
            "status": "error",
            "message": f"Error fetching models: {str(e)}",
            "models": []
        }
```

## Frontend Integration

This FastAPI server provides a simple, well-structured API that any frontend can use. The key endpoints are:

- `POST /api/oblix/initialize` - Set up Oblix with models and API keys
- `POST /api/oblix/create-session` - Create a new chat session
- `POST /api/oblix/stream-chat` - Send messages and get responses

See the [Frontend Integration Guide](frontend-integration.md) for examples of connecting your UI to this API.

## Running in Production

For production deployment, consider:

1. Using a proper ASGI server like Uvicorn behind Nginx
2. Setting up process management with systemd or Docker
3. Implementing proper API authentication
4. Restricting CORS to your frontend domains only
5. Setting up logging to file or a centralized logging system

## Complete Example

For a complete working example of an Oblix FastAPI server, including all recommended features, see our [oblix-fastapi-example](https://github.com/oblix-ai/oblix-fastapi-example) repository.
