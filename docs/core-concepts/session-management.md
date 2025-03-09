# Session Management

Oblix provides built-in session management capabilities to handle conversational interactions with AI models. This page explains how sessions work and how to use them effectively.

## What are Sessions?

Sessions in Oblix are persistent containers for conversation history that:

1. **Store** messages between users and AI models
2. **Maintain** context across multiple interactions
3. **Persist** across application restarts
4. **Enable** stateful conversations

Sessions are particularly useful for building chat applications, virtual assistants, or any interactive AI experience where context matters.

## Session Lifecycle

A typical session lifecycle includes:

1. **Creation** - A new session is created with a unique ID
2. **Message Exchange** - User and AI messages are added to the session
3. **Context Maintenance** - Context is preserved between interactions
4. **Persistence** - Session data is saved to disk
5. **Retrieval** - Sessions can be loaded by ID for continued interaction
6. **Deletion** - Sessions can be deleted when no longer needed

## Working with Sessions

### Creating a Session

```python
from oblix import OblixClient

client = OblixClient(oblix_api_key="your_api_key")

# Create a new session
session_id = await client.create_session(
    title="Customer Support Chat",
    initial_context={"customer_id": "cust_123", "subscription_tier": "premium"}
)

print(f"Created session: {session_id}")
```

### Executing Prompts in a Session

```python
# Set the current session
client.current_session_id = session_id

# Execute prompts in the session context
response = await client.execute("Hello, I'm having trouble with my account")

# The context is automatically maintained
follow_up_response = await client.execute("Can you help me reset my password?")
```

### Using Chat Methods

For chat applications, Oblix provides simplified methods:

```python
# Send a single message in a session
response = await client.chat_once("Hello, how can I help you today?")
session_id = response["session_id"]

# Continue the conversation in the same session
response = await client.chat_once("I need help with my subscription", session_id)
```

### Interactive Chat Sessions

For command-line or interactive applications:

```python
# Start an interactive chat session
await client.start_chat()

# Or resume an existing session
await client.start_chat(session_id="existing_session_id")
```

### Managing Sessions

Oblix provides methods to list, load, and delete sessions:

```python
# List recent sessions
sessions = client.list_sessions(limit=10)
for session in sessions:
    print(f"ID: {session['id']} | Title: {session['title']} | Messages: {session['message_count']}")

# Load a specific session
session_data = client.load_session("session_id_here")

# Delete a session
success = client.delete_session("session_id_here")
```

## Session Storage

By default, Oblix stores sessions as JSON files in a local directory:

- **Default location**: `~/.oblix/sessions/`
- **Format**: Each session is stored as a separate JSON file
- **Naming**: Files are named using the session ID

You can customize the storage location:

```python
from oblix.sessions import SessionManager

# Custom session directory
session_manager = SessionManager(base_dir="/path/to/custom/directory")

# Use with client
client = OblixClient(oblix_api_key="your_api_key")
client.session_manager = session_manager
```

## Session Structure

Each session contains:

- **ID**: Unique identifier for the session
- **Title**: Human-readable title for the session
- **Created/Updated**: Timestamps for creation and last update
- **Messages**: Array of message objects with:
  - **Role**: 'user' or 'assistant'
  - **Content**: Message text
  - **Timestamp**: When the message was added
- **Context**: Optional dictionary for additional context

Example session structure:

```json
{
  "id": "5f3e9a7b-6c1d-4d8e-9e7a-9b8c7d6e5f4a",
  "title": "Technical Support Chat",
  "created_at": "2023-07-15T14:30:22.123Z",
  "updated_at": "2023-07-15T14:35:46.789Z",
  "messages": [
    {
      "id": "msg_1",
      "role": "user",
      "content": "I'm having trouble installing the software.",
      "timestamp": "2023-07-15T14:30:22.123Z"
    },
    {
      "id": "msg_2",
      "role": "assistant",
      "content": "I'm sorry to hear that. What operating system are you using?",
      "timestamp": "2023-07-15T14:30:25.456Z"
    },
    {
      "id": "msg_3",
      "role": "user",
      "content": "I'm using Windows 11.",
      "timestamp": "2023-07-15T14:35:46.789Z"
    }
  ],
  "context": {
    "user_id": "user_789",
    "product": "Oblix SDK"
  }
}
```

## Context Management

Sessions maintain conversation history, which is used as context for future interactions. By default, Oblix includes recent messages when generating responses, allowing models to understand the conversation flow.

You can also provide additional context when creating a session:

```python
# Create session with initial context
session_id = await client.create_session(
    initial_context={
        "user_name": "Alex",
        "preferences": {
            "language": "English",
            "format": "concise"
        }
    }
)
```

## Model Consistency in Sessions

When using sessions with multiple available models, Oblix attempts to maintain consistency by:

1. Using the same model for all messages in a session when possible
2. Falling back to alternative models only when necessary
3. Including sufficient context to maintain conversation flow even when switching models

## Advanced Session Features

### Session Metadata

You can access and update session metadata:

```python
# Get session data
session_data = client.load_session(session_id)

# Access metadata
title = session_data["title"]
created_at = session_data["created_at"]
message_count = len(session_data["messages"])
```

### Message Filtering

When working with long conversations, you can filter messages:

```python
# Get only the most recent messages
recent_messages = session_data["messages"][-5:]

# Filter by role
user_messages = [msg for msg in session_data["messages"] if msg["role"] == "user"]
```

### Context Window Management

For long conversations that might exceed model context windows, Oblix automatically:

1. Selects the most relevant messages as context
2. Prioritizes recent messages
3. Includes critical context from earlier in the conversation

## Best Practices

When working with sessions:

- **Create dedicated sessions** for different conversation types
- **Include relevant initial context** to guide the conversation
- **Clean up old sessions** to manage storage
- **Monitor session length** for very long conversations
- **Consider model consistency** when choosing models for sessions

By effectively using Oblix's session management capabilities, you can build conversational AI applications that maintain context and provide a natural, coherent user experience.

