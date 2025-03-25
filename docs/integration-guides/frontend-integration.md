# Frontend Integration

This guide shows how to integrate an Oblix backend with various frontend technologies to create robust AI-powered applications.

## Overview

After setting up your Oblix FastAPI server as described in the [FastAPI Server Guide](fastapi-server.md), you can connect virtually any frontend technology to it. This guide provides examples for common frontend frameworks.

## API Endpoints

The Oblix FastAPI server exposes these primary endpoints:

- `POST /api/oblix/initialize` - Initialize the Oblix client with API keys and models
- `POST /api/oblix/create-session` - Create a new chat session
- `POST /api/oblix/stream-chat` - Send a message and get a response
- `GET /api/oblix/sessions` - List all available chat sessions
- `POST /api/ollama/status` - Check if Ollama is running
- `POST /api/ollama/models` - List available Ollama models

## React Integration

Here's a simple React component for a chat interface that connects to the Oblix FastAPI server:

```jsx
import React, { useState, useEffect, useRef } from 'react';
import axios from 'axios';

const API_BASE_URL = 'http://localhost:8000/api';

function ChatInterface() {
  const [initialized, setInitialized] = useState(false);
  const [loading, setLoading] = useState(false);
  const [message, setMessage] = useState('');
  const [conversation, setConversation] = useState([]);
  const [sessionId, setSessionId] = useState(null);
  const [modelInfo, setModelInfo] = useState({});
  
  const messageEndRef = useRef(null);

  // Initialize Oblix on component mount
  useEffect(() => {
    const initializeOblix = async () => {
      try {
        setLoading(true);
        const response = await axios.post(`${API_BASE_URL}/oblix/initialize`, {
          oblix_api_key: 'your_oblix_api_key', // Get this from env or user input
          models: [
            {
              model_type: 'OLLAMA',
              model_name: 'llama2',
              endpoint: 'http://localhost:11434'
            },
            {
              model_type: 'OPENAI',
              model_name: 'gpt-3.5-turbo',
              api_key: 'your_openai_api_key' // Get this from env or user input
            }
          ]
        });
        
        if (response.data.status === 'success') {
          setInitialized(true);
          // Create a new session
          const sessionResponse = await axios.post(`${API_BASE_URL}/oblix/create-session`, {
            title: 'New Chat'
          });
          
          if (sessionResponse.data.status === 'success') {
            setSessionId(sessionResponse.data.sessionId);
          }
        }
      } catch (error) {
        console.error('Failed to initialize Oblix:', error);
      } finally {
        setLoading(false);
      }
    };
    
    initializeOblix();
  }, []);
  
  // Scroll to bottom of messages
  useEffect(() => {
    messageEndRef.current?.scrollIntoView({ behavior: 'smooth' });
  }, [conversation]);
  
  const sendMessage = async () => {
    if (!message.trim() || !initialized || loading) return;
    
    try {
      // Add user message to conversation
      setConversation(prev => [...prev, { 
        role: 'user', 
        content: message 
      }]);
      
      setLoading(true);
      setMessage('');
      
      // Send message to Oblix
      const response = await axios.post(`${API_BASE_URL}/oblix/stream-chat`, {
        message: message,
        session_id: sessionId,
        temperature: 0.7
      });
      
      if (response.data.status === 'success') {
        // Add AI response to conversation
        setConversation(prev => [...prev, { 
          role: 'assistant', 
          content: response.data.response 
        }]);
        
        // Store model info for display
        setModelInfo({
          model: response.data.model_id,
          latency: response.data.metrics?.latency,
          routing: response.data.routing_decision
        });
      }
    } catch (error) {
      console.error('Failed to send message:', error);
      // Add error message to conversation
      setConversation(prev => [...prev, { 
        role: 'system', 
        content: `Error: ${error.message || 'Failed to send message'}` 
      }]);
    } finally {
      setLoading(false);
    }
  };
  
  return (
    <div className="chat-container">
      <div className="chat-messages">
        {conversation.map((msg, index) => (
          <div key={index} className={`message ${msg.role}`}>
            {msg.content}
          </div>
        ))}
        {loading && <div className="message assistant loading">Thinking...</div>}
        <div ref={messageEndRef} />
      </div>
      
      <div className="model-info">
        {modelInfo.model && (
          <div>
            Using: {modelInfo.model} | 
            Latency: {modelInfo.latency?.toFixed(0)}ms |
            Routing: {modelInfo.routing?.connectivity_routing?.target || 'default'}
          </div>
        )}
      </div>
      
      <div className="chat-input">
        <input
          type="text"
          value={message}
          onChange={(e) => setMessage(e.target.value)}
          placeholder="Type your message..."
          disabled={!initialized || loading}
          onKeyPress={(e) => e.key === 'Enter' && sendMessage()}
        />
        <button 
          onClick={sendMessage}
          disabled={!initialized || loading || !message.trim()}
        >
          Send
        </button>
      </div>
    </div>
  );
}

export default ChatInterface;
```

## Vue.js Integration

Here's a Vue.js component for integrating with Oblix:

```vue
<template>
  <div class="chat-container">
    <div class="chat-messages">
      <div 
        v-for="(msg, index) in conversation" 
        :key="index" 
        :class="['message', msg.role]"
      >
        {{ msg.content }}
      </div>
      <div v-if="loading" class="message assistant loading">
        Thinking...
      </div>
      <div ref="messageEnd"></div>
    </div>
    
    <div class="model-info" v-if="modelInfo.model">
      <div>
        Using: {{ modelInfo.model }} | 
        Latency: {{ modelInfo.latency ? modelInfo.latency.toFixed(0) : 0 }}ms |
        Routing: {{ modelInfo.routing?.connectivity_routing?.target || 'default' }}
      </div>
    </div>
    
    <div class="chat-input">
      <input
        type="text"
        v-model="message"
        placeholder="Type your message..."
        :disabled="!initialized || loading"
        @keyup.enter="sendMessage"
      />
      <button 
        @click="sendMessage"
        :disabled="!initialized || loading || !message.trim()"
      >
        Send
      </button>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'ChatInterface',
  data() {
    return {
      initialized: false,
      loading: false,
      message: '',
      conversation: [],
      sessionId: null,
      modelInfo: {},
      apiBaseUrl: 'http://localhost:8000/api'
    };
  },
  mounted() {
    this.initializeOblix();
  },
  updated() {
    this.scrollToBottom();
  },
  methods: {
    async initializeOblix() {
      try {
        this.loading = true;
        
        const response = await axios.post(`${this.apiBaseUrl}/oblix/initialize`, {
          oblix_api_key: 'your_oblix_api_key', // Get this from env or user input
          models: [
            {
              model_type: 'OLLAMA',
              model_name: 'llama2',
              endpoint: 'http://localhost:11434'
            },
            {
              model_type: 'OPENAI',
              model_name: 'gpt-3.5-turbo',
              api_key: 'your_openai_api_key' // Get this from env or user input
            }
          ]
        });
        
        if (response.data.status === 'success') {
          this.initialized = true;
          
          // Create a new session
          const sessionResponse = await axios.post(`${this.apiBaseUrl}/oblix/create-session`, {
            title: 'New Chat'
          });
          
          if (sessionResponse.data.status === 'success') {
            this.sessionId = sessionResponse.data.sessionId;
          }
        }
      } catch (error) {
        console.error('Failed to initialize Oblix:', error);
      } finally {
        this.loading = false;
      }
    },
    async sendMessage() {
      if (!this.message.trim() || !this.initialized || this.loading) return;
      
      try {
        // Add user message to conversation
        this.conversation.push({ 
          role: 'user', 
          content: this.message 
        });
        
        const userMessage = this.message;
        this.message = '';
        this.loading = true;
        
        // Send message to Oblix
        const response = await axios.post(`${this.apiBaseUrl}/oblix/stream-chat`, {
          message: userMessage,
          session_id: this.sessionId,
          temperature: 0.7
        });
        
        if (response.data.status === 'success') {
          // Add AI response to conversation
          this.conversation.push({ 
            role: 'assistant', 
            content: response.data.response 
          });
          
          // Store model info for display
          this.modelInfo = {
            model: response.data.model_id,
            latency: response.data.metrics?.latency,
            routing: response.data.routing_decision
          };
        }
      } catch (error) {
        console.error('Failed to send message:', error);
        // Add error message to conversation
        this.conversation.push({ 
          role: 'system', 
          content: `Error: ${error.message || 'Failed to send message'}` 
        });
      } finally {
        this.loading = false;
      }
    },
    scrollToBottom() {
      if (this.$refs.messageEnd) {
        this.$refs.messageEnd.scrollIntoView({ behavior: 'smooth' });
      }
    }
  }
};
</script>

<style scoped>
.chat-container {
  display: flex;
  flex-direction: column;
  height: 500px;
  border: 1px solid #ccc;
  border-radius: 8px;
}

.chat-messages {
  flex: 1;
  overflow-y: auto;
  padding: 15px;
}

.message {
  margin-bottom: 10px;
  padding: 10px;
  border-radius: 8px;
  max-width: 70%;
}

.user {
  background-color: #e1f5fe;
  align-self: flex-end;
  margin-left: auto;
}

.assistant {
  background-color: #f1f1f1;
  align-self: flex-start;
}

.system {
  background-color: #ffebee;
  align-self: center;
  text-align: center;
  width: 100%;
}

.model-info {
  font-size: 12px;
  color: #666;
  padding: 5px 15px;
  border-top: 1px solid #eee;
}

.chat-input {
  display: flex;
  padding: 15px;
  border-top: 1px solid #eee;
}

input {
  flex: 1;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  margin-right: 10px;
}

button {
  padding: 10px 15px;
  background-color: #2196f3;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

button:disabled {
  background-color: #cccccc;
  cursor: not-allowed;
}

.loading {
  opacity: 0.7;
  font-style: italic;
}
</style>
```

## Electron Integration

For desktop applications using Electron, you can integrate with Oblix in two ways:

1. **HTTP API Integration**: Use the FastAPI server running separately, similar to web frontend integration
2. **Embedded Integration**: Run the FastAPI server directly within the Electron app

### Embedded FastAPI Server in Electron

Here's how to embed the Oblix FastAPI server directly in an Electron application:

```javascript
// electron/main.js
const { app, BrowserWindow } = require('electron');
const { spawn } = require('child_process');
const path = require('path');
const fs = require('fs');
const portfinder = require('portfinder');

let mainWindow;
let backendProcess;
let backendPort;

async function startBackend() {
  try {
    // Find a free port
    backendPort = await portfinder.getPortPromise({
      port: 8000,
      stopPort: 9000
    });
    
    // Path to Python backend script
    const backendPath = path.join(__dirname, 'backend', 'oblix_fastapi_server.py');
    
    // Check if the backend file exists
    if (!fs.existsSync(backendPath)) {
      console.error('Backend file not found:', backendPath);
      app.quit();
      return;
    }
    
    // Start the backend process
    backendProcess = spawn('python', [backendPath, `--port=${backendPort}`], {
      stdio: 'pipe'
    });
    
    // Handle backend startup
    return new Promise((resolve, reject) => {
      // Set a timeout for backend startup
      const timeout = setTimeout(() => {
        reject(new Error('Backend startup timed out'));
      }, 10000);
      
      // Listen for backend ready signal in stdout
      backendProcess.stdout.on('data', (data) => {
        const output = data.toString();
        console.log('Backend:', output);
        
        // Check for server started message
        if (output.includes('Application startup complete') || 
            output.includes(`running on http://127.0.0.1:${backendPort}`)) {
          clearTimeout(timeout);
          resolve(backendPort);
        }
      });
      
      // Listen for errors
      backendProcess.stderr.on('data', (data) => {
        console.error('Backend error:', data.toString());
      });
      
      backendProcess.on('error', (error) => {
        clearTimeout(timeout);
        console.error('Failed to start backend:', error);
        reject(error);
      });
      
      backendProcess.on('exit', (code) => {
        if (code !== 0 && code !== null) {
          clearTimeout(timeout);
          reject(new Error(`Backend exited with code ${code}`));
        }
      });
    });
  } catch (error) {
    console.error('Error starting backend:', error);
    throw error;
  }
}

async function createWindow() {
  try {
    // Start the backend first
    await startBackend();
    
    // Create the browser window
    mainWindow = new BrowserWindow({
      width: 1200,
      height: 800,
      webPreferences: {
        nodeIntegration: true,
        contextIsolation: false
      }
    });
    
    // Pass the backend port to the renderer
    global.backendPort = backendPort;
    
    // Load the app
    if (app.isPackaged) {
      mainWindow.loadFile(path.join(__dirname, 'dist', 'index.html'));
    } else {
      mainWindow.loadURL('http://localhost:3000'); // Development server
    }
    
    // Open DevTools in development
    if (!app.isPackaged) {
      mainWindow.webContents.openDevTools();
    }
    
    mainWindow.on('closed', () => {
      mainWindow = null;
    });
  } catch (error) {
    console.error('Failed to create window:', error);
    app.quit();
  }
}

app.on('ready', createWindow);

app.on('window-all-closed', () => {
  if (process.platform !== 'darwin') {
    app.quit();
  }
});

app.on('activate', () => {
  if (mainWindow === null) {
    createWindow();
  }
});

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

## Mobile Integration (React Native)

For React Native apps, you can connect to your Oblix FastAPI server using the Fetch API or Axios:

```javascript
// api.js
import axios from 'axios';

const API_BASE_URL = 'http://your-oblix-api-server:8000/api';

export const oblixApi = {
  initialize: async (oblixApiKey, models) => {
    try {
      const response = await axios.post(`${API_BASE_URL}/oblix/initialize`, {
        oblix_api_key: oblixApiKey,
        models
      });
      return response.data;
    } catch (error) {
      console.error('Oblix initialization error:', error);
      throw error;
    }
  },
  
  createSession: async (title = 'New Chat') => {
    try {
      const response = await axios.post(`${API_BASE_URL}/oblix/create-session`, {
        title
      });
      return response.data;
    } catch (error) {
      console.error('Create session error:', error);
      throw error;
    }
  },
  
  sendMessage: async (message, sessionId, temperature = 0.7) => {
    try {
      const response = await axios.post(`${API_BASE_URL}/oblix/stream-chat`, {
        message,
        session_id: sessionId,
        temperature
      });
      return response.data;
    } catch (error) {
      console.error('Send message error:', error);
      throw error;
    }
  },
  
  listSessions: async () => {
    try {
      const response = await axios.post(`${API_BASE_URL}/oblix/sessions`);
      return response.data;
    } catch (error) {
      console.error('List sessions error:', error);
      throw error;
    }
  }
};
```

## Session Management Best Practices

When implementing a frontend that works with Oblix sessions, follow these best practices:

### Initialization Sequence

Always follow the proper initialization sequence:

```javascript
// 1. First initialize Oblix client with models
const initResult = await api.post('/oblix/initialize', {
  oblix_api_key: apiKey,
  models: [
    { model_type: "OLLAMA", model_name: "llama2" },
    { model_type: "OPENAI", model_name: "gpt-4" }
  ]
});

// 2. Then create a session
const sessionResult = await api.post('/oblix/create-session', {
  title: 'New Chat'
});
const sessionId = sessionResult.data.sessionId;

// 3. Now you can send messages
const response = await api.post('/oblix/stream-chat', {
  message: "Hello!",
  session_id: sessionId
});
```

### Switching Between Sessions

When switching between chat sessions, ensure you send the correct session ID with each request:

```javascript
// When user clicks on a different chat
const switchToSession = async (sessionId) => {
  // Simply include the session_id in subsequent requests
  // The backend will handle context preservation
  const response = await api.post('/oblix/stream-chat', {
    message: "Continuing our conversation...",
    session_id: sessionId
  });
};
```

### Error Handling

Handle common session-related errors:

```javascript
const sendMessage = async (message, sessionId) => {
  try {
    const response = await api.post('/oblix/stream-chat', {
      message,
      session_id: sessionId
    });
    return response.data;
  } catch (error) {
    // Check for specific error types
    if (error.response?.data?.message?.includes('not initialized')) {
      // Oblix client not initialized - reinitialize and try again
      await initializeOblix();
      return sendMessage(message, sessionId);
    } else if (error.response?.status === 404) {
      // Session not found - create a new one
      const newSession = await createSession('New Chat');
      return sendMessage(message, newSession.sessionId);
    } else {
      // Handle other errors
      console.error('Message error:', error);
      throw error;
    }
  }
};
```

## Tips for a Better User Experience

1. **Loading States**: Always show loading indicators during API calls
2. **Error Handling**: Implement comprehensive error handling, especially for connectivity issues
3. **Model Selection**: Allow users to choose which models they want to use
4. **Debug Mode**: Add a debug toggle to display detailed metrics and routing decisions
5. **Session Management**: Provide a way to view, select, and delete chat sessions
6. **Offline Detection**: Show appropriate messages when working in offline mode
7. **Progress Indicators**: For long-running operations, show meaningful progress indicators

## Security Considerations

When building a frontend that integrates with Oblix:

1. **API Key Management**: Never store API keys in client-side code
2. **Authentication**: Implement proper authentication for your FastAPI server
3. **Rate Limiting**: Add rate limiting to prevent abuse
4. **CORS Configuration**: Limit CORS to only your frontend domains in production
5. **TLS/SSL**: Always use HTTPS in production

## Next Steps

- Check the [Troubleshooting Guide](troubleshooting.md) for common issues and solutions
- Explore advanced features like streaming responses in the [FastAPI Server Guide](fastapi-server.md)
- For production deployments, review security and performance optimization tips